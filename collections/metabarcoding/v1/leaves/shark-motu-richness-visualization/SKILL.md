---
name: shark-motu-richness-visualization
description: Use when a MOTU-by-sample read count table (e.g. from an OBITools pipeline
  with sumaclust 99% clustering) must be converted to per-location species accumulation
  curves to judge sampling completeness.
when_to_use_negative:
- The input is raw FASTQ reads or an upstream OBITools stage (quality filtering, demultiplexing,
  chimera removal) rather than a clustered MOTU-by-sample table
- The goal is testing differences in MOTU richness/read abundance among locations
  (the article uses a generalized linear model for that, not accumulation curves)
- Read-abundance patterns are of interest — the curves use presence/absence detection
  per sample, not read counts
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Host language for converting the read table to presence/absence, permuting
    sample order, and rendering the Fig. 4 accumulation-curve layout (custom scripts
    at github.com/metabarpark)
  repo: github.com/metabarpark
- name: vegan
  role: specaccum() computes the sample-based species accumulation curves with 100
    random permutations per location
- name: sumaclust
  role: Upstream MOTU delimitation at a 99% similarity threshold that defines the
    richness units plotted in the curves
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:eff6f0ecd603533a126d90c75f89f8d06aa63e39272d4754abfc9737e9d562eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/metabarpark
  tools:
  - R
  - sumaclust
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# shark-motu-richness-visualization

## Summary

Computes and plots sample-based species accumulation curves of elasmobranch MOTUs per location (Caribbean and New Caledonia) using vegan::specaccum with 100 random permutations, to test whether eDNA metabarcoding sampling captured total shark diversity or saturated.

## When to use

Use when a MOTU-by-sample read count table (e.g. from an OBITools pipeline with sumaclust 99% clustering) must be converted to per-location species accumulation curves to judge sampling completeness.

## When NOT to use

- The input is raw FASTQ reads or an upstream OBITools stage (quality filtering, demultiplexing, chimera removal) rather than a clustered MOTU-by-sample table
- The goal is testing differences in MOTU richness/read abundance among locations (the article uses a generalized linear model for that, not accumulation curves)
- Read-abundance patterns are of interest — the curves use presence/absence detection per sample, not read counts

## Inputs

- MOTU-by-sample read count table (OBITools output; MOTUs clustered with sumaclust at 99% similarity, single-read MOTUs removed) from github.com/metabarpark or data.mendeley.com/datasets/489gckxbz3/1
- Sample metadata mapping each sample to its sampling location (Caribbean and New Caledonian sites)

## Outputs

- Per-location sample-based species accumulation curves (vegan::specaccum objects with 100 permutations)
- Combined Fig. 4-style figure (species_accumulation_curves.png) showing all location curves with MOTU richness vs. number of samples

## How to apply

Load the MOTU-by-sample read count table produced by the OBITools pipeline (MOTUs delimited with sumaclust at a 99% similarity threshold, all single-read MOTUs removed) from the custom R-scripts repository (github.com/metabarpark) or the Mendeley deposit (data.mendeley.com/datasets/489gckxbz3/1). Convert read counts to presence/absence MOTU detection per sample and group samples by location, excluding Belize from the curve set when reproducing the single-shark-MOTU Fig. 4 layout. Compute one sample-based accumulation curve per location with vegan::specaccum using 100 random permutations of sample order. Plot all location curves together (R base graphics or ggplot2) and inspect whether any curve reaches a plateau; the expected pattern is that no Caribbean or New Caledonian curve plateaus, with Caribbean curves flattening after ~10 samples except the Bahamas. A non-plateauing curve is the article's evidence that sampling did not capture total elasmobranch diversity, so absence of saturation is the correct result, not a pipeline failure.

## Related tools

- **R** (Host language for converting the read table to presence/absence, permuting sample order, and rendering the Fig. 4 accumulation-curve layout (custom scripts at github.com/metabarpark)) — github.com/metabarpark
- **vegan** (specaccum() computes the sample-based species accumulation curves with 100 random permutations per location)
- **sumaclust** (Upstream MOTU delimitation at a 99% similarity threshold that defines the richness units plotted in the curves)

## Examples

```
library(vegan); sac <- specaccum(motu_pa[, loc_samples], method = "random", permutations = 100); plot(sac)
```

## Evaluation signals

- species_accumulation_curves.png exists and matches the Fig. 4 reference layout in curve count and axes
- No Caribbean or New Caledonian curve reaches a plateau in MOTU richness
- Caribbean curves visually flatten after ~10 samples, except the Bahamas curve
- Each curve is computed from 100 random permutations of sample order (vegan::specaccum)
- Belize is excluded from the curve set and all single-read MOTUs have been removed before richness computation

## Limitations

- Non-saturation means total elasmobranch diversity was not captured; the curves indicate sampling effort needed, not absolute species counts
- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species (e.g. Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple species in BOLD, so MOTU richness is not exactly species richness
- Removing all single-read MOTUs is a conservative contamination/false-positive control that may understate richness

## Evidence

- [results] Species accumulation curves do not reach a plateau in MOTU richness for any Caribbean or New Caledonian location (Fig. 4): [evidence span withheld — anchor status: not_found]
- [methods] Compute sample-based species accumulation curves per location with vegan::specaccum, using 100 random permutations of sample order per location.: [evidence span withheld — anchor status: not_found]
- [discussion] Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance among our samples: "Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance among our samples"
- [discussion] we have opted for a more conservative approach and have removed all single read MOTUs from our samples.: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [readme] Reference databases in fasta format for Eukaryotic metabarcoding: "Reference databases in fasta format for Eukaryotic metabarcoding"
