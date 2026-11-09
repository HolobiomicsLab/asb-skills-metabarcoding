---
name: species-accumulation-curve-estimation
description: Use when you have a MOTU-by-sample detection table from eDNA metabarcoding
  (sumaclust 99% clustering, single-read MOTUs removed) and need to assess whether
  sample-based richness reaches a plateau per location, as in Fig. 4.
when_to_use_negative:
- Input is still raw FASTQ or unclustered reads — MOTUs must first be delimited with
  sumaclust at 99% similarity and single-read MOTUs removed before curves are meaningful
- Goal is testing differences in MOTU richness or read abundance among locations with
  a generalized linear model — use the paper's GLM analysis instead of accumulation
  curves
- Richness already saturates or the sampling design has too few samples per location
  to support 100-permutation resampling
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: vegan
  role: R package whose specaccum function computes sample-based species accumulation
    curves with random permutations of sample order
- name: R (custom metabarpark scripts)
  role: Hosts the custom R scripts for converting the MOTU read table to presence/absence,
    running specaccum per location, and plotting the Fig. 4 layout
  repo: http://github.com/metabarpark
- name: sumaclust
  role: Upstream MOTU clustering tool that produced the 99%-similarity MOTU table
    consumed by this skill
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:cbf0672e45afa2b875c106c4a7229f40e6799fed33413d567337cad72246d869
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R (custom metabarpark scripts)
  - sumaclust
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# species-accumulation-curve-estimation

## Summary

Computes sample-based species accumulation curves over MOTU-by-sample presence/absence data to test whether elasmobranch MOTU richness saturates with sampling effort at Caribbean and New Caledonian eDNA sampling locations.

## When to use

Use when you have a MOTU-by-sample detection table from eDNA metabarcoding (sumaclust 99% clustering, single-read MOTUs removed) and need to assess whether sample-based richness reaches a plateau per location, as in Fig. 4.

## When NOT to use

- Input is still raw FASTQ or unclustered reads — MOTUs must first be delimited with sumaclust at 99% similarity and single-read MOTUs removed before curves are meaningful
- Goal is testing differences in MOTU richness or read abundance among locations with a generalized linear model — use the paper's GLM analysis instead of accumulation curves
- Richness already saturates or the sampling design has too few samples per location to support 100-permutation resampling

## Inputs

- MOTU-by-sample read count table (OBITools/sumaclust 99% output, single-read MOTUs removed)
- Sample-to-location metadata mapping (Caribbean and New Caledonian locations)
- MOTU taxonomic assignment table (ecotag assignments against the bespoke elasmobranch COI reference database)

## Outputs

- Per-location sample-based species accumulation curves (richness vs. number of samples) from vegan::specaccum
- Fig. 4-style composite figure (species_accumulation_curves.png) showing non-saturating richness curves per location

## How to apply

Start from the MOTU-by-sample read count table produced by the OBITools pipeline (MOTUs delimited with sumaclust at a 99% similarity threshold, all single-read MOTUs removed), obtained from the metabarpark custom R-scripts repository or the Mendeley deposit (https://data.mendeley.com/datasets/489gckxbz3/1). Convert read counts to presence/absence detection per sample and group samples by location, excluding Belize from the curve set. Per location, compute sample-based species accumulation curves with vegan::specaccum using the 'random' method with 100 random permutations of sample order. Plot all location-specific curves together in the Fig. 4 layout (single shark MOTU dataset) using R base graphics or ggplot2. The key decision is visual: confirm that no Caribbean or New Caledonian curve reaches a plateau, i.e. increased sampling would still add new elasmobranch MOTUs.

## Related tools

- **vegan** (R package whose specaccum function computes sample-based species accumulation curves with random permutations of sample order)
- **R (custom metabarpark scripts)** (Hosts the custom R scripts for converting the MOTU read table to presence/absence, running specaccum per location, and plotting the Fig. 4 layout) — http://github.com/metabarpark
- **sumaclust** (Upstream MOTU clustering tool that produced the 99%-similarity MOTU table consumed by this skill)

## Examples

```
R> sac <- vegan::specaccum(motu_pa, method = "random", permutations = 100); plot(sac, xlab = "Samples", ylab = "MOTU richness")
```

## Evaluation signals

- No Caribbean or New Caledonian accumulation curve reaches a plateau in MOTU richness
- Caribbean curves flatten after ~10 samples except the Bahamas curve, which continues rising
- Belize is excluded from the plotted curve set, matching the Fig. 4 layout (curve count and axes)
- Curves are computed from a presence/absence matrix, not read counts, after removal of all single-read MOTUs
- Each specaccum run uses 100 random permutations of sample order per location

## Limitations

- Sampling did not capture total elasmobranch diversity — curves are non-saturating, so absolute richness is underestimated
- Single-read MOTUs were removed as a conservative contamination/false-positive control, so very rare detections are absent from the curves
- Taxonomic resolution of the 127 bp COI fragment cannot separate closely related species (e.g. Carcharhinus, Rhizoprionodon, Negaprion), affecting MOTU-level richness counts
- Some elasmobranch species present at sampling sites could not be detected due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered)

## Evidence

- [discussion] sumaclust 99% clustering defines the MOTU table: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] single-read MOTUs removed conservatively: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] custom R scripts availability: "All custom R scripts are publicly available from http://github.com/metabarpark."
- [discussion] original sequences deposit: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1."
- [readme] README describes reference database deposit: "Reference databases in fasta format for Eukaryotic metabarcoding"
