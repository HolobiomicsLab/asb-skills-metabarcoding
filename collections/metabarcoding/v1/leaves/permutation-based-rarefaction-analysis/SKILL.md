---
name: permutation-based-rarefaction-analysis
description: Use when you have a MOTU-by-sample detection table from metabarcoding
  and need to test whether local taxonomic richness has saturated or whether more
  sampling effort would recover additional diversity.
when_to_use_negative:
- You only have read-abundance summaries per location without sample-level detection
  — the curves require individual sample order permutations.
- The aim is testing richness differences among locations (use the paper's GLM approach
  on MOTU richness and square-root transformed read abundances instead).
- MOTUs have not been filtered (e.g. single-read MOTUs retained), which would inflate
  rarefaction richness with false positives.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: vegan (specaccum)
  role: Computes sample-based species accumulation curves with random sample-order
    permutations per location in R
- name: sumaclust
  role: Upstream MOTU delimitation at a constant 99% similarity threshold that defines
    the richness units fed into the accumulation curves
- name: R
  role: Environment for presence/absence conversion, permutation rarefaction, and
    plotting the Fig. 4 curves
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:76f79d6501db01caa4d7ec1d7b5a32676d8274a1cbbfc7c638b672ecc173924c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - sumaclust
  - vegan (specaccum)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# permutation-based-rarefaction-analysis

## Summary

Assesses whether eDNA MOTU richness is fully sampled by computing sample-based rarefaction (species accumulation) curves with random permutations, revealing that Caribbean and New Caledonian shark MOTU richness does not plateau. It quantifies sampling sufficiency from presence/absence detection per sample.

## When to use

Use when you have a MOTU-by-sample detection table from metabarcoding and need to test whether local taxonomic richness has saturated or whether more sampling effort would recover additional diversity.

## When NOT to use

- You only have read-abundance summaries per location without sample-level detection — the curves require individual sample order permutations.
- The aim is testing richness differences among locations (use the paper's GLM approach on MOTU richness and square-root transformed read abundances instead).
- MOTUs have not been filtered (e.g. single-read MOTUs retained), which would inflate rarefaction richness with false positives.

## Inputs

- MOTU-by-sample read count table from the OBITools pipeline (sumaclust 99% threshold, single-read MOTUs removed), from github.com/metabarpark or https://data.mendeley.com/datasets/489gckxbz3/1
- Sample-to-location metadata (Caribbean and New Caledonian/Coral Sea sites)

## Outputs

- Location-specific sample-based species accumulation curves (mean richness ± permutation spread)
- species_accumulation_curves.png matching the Fig. 4 layout (curve count, axes)
- Qualitative assessment of plateau vs. non-plateau per location

## How to apply

Convert the read-count table (MOTUs delimited with sumaclust at a 99% similarity threshold, single-read MOTUs removed) into presence/absence MOTU detection per sample, then group samples by location (Belize excluded in the reference figure). Compute sample-based accumulation curves per location with vegan::specaccum using 100 random permutations of sample order. Plot all location curves together (Fig. 4 layout) and inspect whether each reaches a plateau; flattening after ~10 samples except the Bahamas indicates under-sampling, so a non-plateau means total diversity was not captured.

## Related tools

- **vegan (specaccum)** (Computes sample-based species accumulation curves with random sample-order permutations per location in R)
- **sumaclust** (Upstream MOTU delimitation at a constant 99% similarity threshold that defines the richness units fed into the accumulation curves)
- **R** (Environment for presence/absence conversion, permutation rarefaction, and plotting the Fig. 4 curves)

## Examples

```
library(vegan); sac <- specaccum(presence_absence_matrix, method="random", permutations=100); plot(sac)
```

## Evaluation signals

- species_accumulation_curves.png exists and matches the Fig. 4 reference layout (number of curves and axes, Belize excluded, single shark MOTU dataset)
- Each curve is based on 100 random permutations of sample order per location (vegan::specaccum default 'random' method)
- MOTU detection table derived from the 99%-threshold sumaclust clustering with all single-read MOTUs removed
- Rendered curves visually reproduce the reported non-plateauing pattern for the Caribbean and New Caledonia locations, with Caribbean curves flattening after ~10 samples except the Bahamas
- No plateau is claimed for any location unless the plotted mean richness clearly levels off at the maximum sample count

## Limitations

- Non-plateauing curves mean total elasmobranch diversity was not captured; the method does not estimate how many species remain undetected.
- MOTU richness depends on the 99% sumaclust threshold and on the conservative removal of single-read MOTUs (contamination/sequencing error cannot be fully excluded), which shapes the rarefaction input.
- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species (some MOTUs 100% identical to multiple BOLD species), so curve 'species' counts may under- or over-split real species.
- Primer mismatches mean some species present in the environment (e.g. visually observed Ginglymostoma cirratum) yield no MOTUs and are invisible to rarefaction.

## Evidence

- [discussion] MOTU clustering threshold: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] single-read MOTU removal: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] conservative contamination control rationale: "it is not possible to completely exclude contamination (or sequencing error) as the potential cause of MOTUs containing only a single read"
- [discussion] custom R scripts availability: "All custom R scripts are publicly available from http://github.com/metabarpark."
- [readme] README citation scope: "Reference databases in fasta format for Eukaryotic metabarcoding"
