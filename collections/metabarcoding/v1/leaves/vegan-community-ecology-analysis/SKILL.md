---
name: vegan-community-ecology-analysis
description: Use when you have a MOTU-by-sample detection table from metabarcoding
  (presence/absence per sample) grouped by location and need to assess whether sampling
  effort captured total species richness via accumulation curves.
when_to_use_negative:
- Input is still raw Illumina reads requiring OBITools preprocessing (illuminapairedend,
  ngsfilter, obigrep 120-135 bp, obiuniq, vsearch uchime-denovo) — run the metabarcoding
  pipeline first
- The goal is testing differences in MOTU richness or read abundance across locations
  rather than sampling-completeness — the paper used a generalized linear model on
  MOTU richness and square-root-transformed read abundances for that
- Input data are presence/absence already aggregated or abundance-based indices are
  required; specaccum here is run on presence/absence detection per sample, not read
  counts
tools:
- name: vegan
  role: R package used to compute sample-based species accumulation curves per location
    with specaccum (100 random permutations of sample order)
- name: R
  role: Analysis environment for the custom scripts converting the read table to presence/absence,
    running specaccum, and rendering the Fig. 4 curve plot
  repo: http://github.com/metabarpark
- name: sumaclust
  role: Upstream MOTU delimiter producing the 99%-similarity MOTU table that feeds
    the accumulation-curve analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:59046fc4e35ac5ca4a48a5fd908f2a555c0a956f7d9c53eb7ffc96317d069988
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - sumaclust
  - vegan
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# vegan-community-ecology-analysis

## Summary

Compute sample-based species accumulation curves over eDNA metabarcoding MOTU detection tables (e.g. elasmobranch MOTUs delimited with sumaclust at 99% similarity) to test whether sampled richness has saturated. The paper used vegan::specaccum in R to show that Caribbean and New Caledonian curves do not plateau, implying unsampled diversity.

## When to use

Use when you have a MOTU-by-sample detection table from metabarcoding (presence/absence per sample) grouped by location and need to assess whether sampling effort captured total species richness via accumulation curves.

## When NOT to use

- Input is still raw Illumina reads requiring OBITools preprocessing (illuminapairedend, ngsfilter, obigrep 120-135 bp, obiuniq, vsearch uchime-denovo) — run the metabarcoding pipeline first
- The goal is testing differences in MOTU richness or read abundance across locations rather than sampling-completeness — the paper used a generalized linear model on MOTU richness and square-root-transformed read abundances for that
- Input data are presence/absence already aggregated or abundance-based indices are required; specaccum here is run on presence/absence detection per sample, not read counts

## Inputs

- MOTU-by-sample read count table from the OBITools/sumaclust (99%) pipeline, with single-read MOTUs removed (Mendeley deposit https://data.mendeley.com/datasets/489gckxbz3/1 or github.com/metabarpark scripts)
- Sample metadata mapping each sample to its sampling location (Caribbean and New Caledonian sites)
- Location exclusion list and dataset scope flags (e.g. exclude Belize; single shark MOTU dataset)

## Outputs

- Per-location sample-based species accumulation curves (vegan::specaccum objects with 100 permutations)
- Combined multi-location accumulation curve figure (species_accumulation_curves.png, Fig. 4 layout)
- Qualitative verdict per location: plateau reached vs. non-saturation of MOTU richness

## How to apply

Load the MOTU-by-sample read count table produced by the OBITools pipeline (sumaclust 99% similarity clustering, single-read MOTUs removed) from the custom R-scripts repository (github.com/metabarpark) or the Mendeley deposit (data.mendeley.com/datasets/489gckxbz3/1). Convert read counts to presence/absence MOTU detection per sample and group samples by location, excluding locations flagged in the target figure (here Belize) and restricting to the single shark MOTU dataset. Compute sample-based species accumulation curves per location with vegan::specaccum using 100 random permutations of sample order per location. Plot all location-specific curves together (Fig. 4 layout) with R base graphics or ggplot2 and inspect whether any curve reaches a plateau; a non-saturating curve indicates more sampling effort would be required to capture total diversity.

## Related tools

- **vegan** (R package used to compute sample-based species accumulation curves per location with specaccum (100 random permutations of sample order))
- **R** (Analysis environment for the custom scripts converting the read table to presence/absence, running specaccum, and rendering the Fig. 4 curve plot) — http://github.com/metabarpark
- **sumaclust** (Upstream MOTU delimiter producing the 99%-similarity MOTU table that feeds the accumulation-curve analysis)

## Examples

```
Rscript -e 'library(vegan); sac <- specaccum(t(motu_pa), method="sample", permutations=100); plot(sac)'
```

## Evaluation signals

- species_accumulation_curves.png exists and matches the Fig. 4 reference layout (same curve count and axes, Belize excluded, single shark MOTU dataset)
- Each curve is computed from 100 random permutations of sample order (specaccum permutations=100)
- Input table is the single-read-MOTU-filtered, sumaclust 99% MOTU set converted to presence/absence per sample before curve computation
- Rendered curves visually reproduce the reported non-plateauing pattern: no Caribbean or New Caledonian curve reaches a plateau, Caribbean curves flattening after ~10 samples except the Bahamas
- Dataset lineage traces back to the Mendeley deposit 489gckxbz3/1 or the metabarpark custom R-scripts repository

## Limitations

- Non-saturating accumulation curves mean total elasmobranch diversity was not captured; conclusions are about sampling completeness, not absolute richness
- The 127 bp COI fragment cannot separate closely related Carcharhinus, Rhizoprionodon and Negaprion species; some MOTUs are 100% identical to multiple BOLD species, so curve 'species' counts are MOTU-level, not species-level
- Some elasmobranchs present at sampling could not be detected due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered), biasing the richness underlying the curves

## Evidence

- [methods] MOTUs delimited with sumaclust at 99% similarity; single-read MOTUs removed upstream of curve computation: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] Custom R scripts (including accumulation-curve analysis) are publicly available: "All custom R scripts are publicly available from http://github.com/metabarpark."
- [discussion] Original filtered sequences underlying the MOTU table are deposited on Mendeley: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1."
- [discussion] Single-read MOTUs removed as conservative false-positive control before downstream analyses: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
