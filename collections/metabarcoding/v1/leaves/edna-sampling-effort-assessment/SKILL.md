---
name: edna-sampling-effort-assessment
description: Use when you have a per-sample eDNA MOTU detection table (e.g. sumaclust
  99% MOTUs) grouped by location and need to determine whether richness is saturated
  or under-sampled.
when_to_use_negative:
- The input is already a curated presence/absence site-by-species matrix for which
  saturation has already been assessed — re-running curves adds no information
- The goal is taxonomic assignment or read-based abundance modelling (e.g. the paper's
  generalized linear model of MOTU richness and square-root-transformed read abundances)
  rather than sampling-effort/saturation assessment
- Samples cannot be resolved to individual sampling units per location (e.g. only
  pooled or per-library reads are available), since sample-based accumulation curves
  require per-sample detection data
tools:
- name: vegan
  role: R package used to compute sample-based species accumulation curves per location
    via specaccum with 100 random permutations of sample order
- name: R
  role: Language in which the MOTU table is converted to presence/absence, curves
    are computed, and the Fig. 4 figure is rendered; custom scripts from the metabarpark
    repository
  repo: http://github.com/metabarpark
- name: sumaclust
  role: Upstream MOTU delimitation step (99% similarity threshold) that produces the
    MOTU set feeding the accumulation curves
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:3b0769ef7f0e1bbd222c15489857afcb03bb0a7637308742cc781cab5fe49bb5
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
yes_rate: 0.8333333333333334
n_claims: 6
---

# edna-sampling-effort-assessment

## Summary

Assess whether sample-based eDNA metabarcoding captured total species diversity by computing species accumulation curves per sampling location from a MOTU-by-sample table and inspecting them for a plateau. This tells the analyst whether increased sampling effort would still add new MOTUs.

## When to use

Use when you have a per-sample eDNA MOTU detection table (e.g. sumaclust 99% MOTUs) grouped by location and need to determine whether richness is saturated or under-sampled.

## When NOT to use

- The input is already a curated presence/absence site-by-species matrix for which saturation has already been assessed — re-running curves adds no information
- The goal is taxonomic assignment or read-based abundance modelling (e.g. the paper's generalized linear model of MOTU richness and square-root-transformed read abundances) rather than sampling-effort/saturation assessment
- Samples cannot be resolved to individual sampling units per location (e.g. only pooled or per-library reads are available), since sample-based accumulation curves require per-sample detection data

## Inputs

- MOTU-by-sample read count table from the OBITools pipeline (sumaclust 99% MOTUs, single-read MOTUs removed), e.g. from the metabarpark R-scripts repository or Mendeley Data deposit 489gckxbz3/1
- Sample metadata mapping each sample to a sampling location (Caribbean and New Caledonian sites, Belize flagged for exclusion)

## Outputs

- Per-location sample-based species accumulation curves (richness vs. number of samples, with 100 random permutations)
- Combined multi-location accumulation-curve figure reproducing the Fig. 4 layout (e.g. species_accumulation_curves.png)
- Qualitative saturation assessment: which locations' curves plateau vs. which indicate unsampled residual diversity

## How to apply

Start from the MOTU-by-sample read count table produced by the OBITools pipeline, where MOTUs were delimited with sumaclust at a constant 99% similarity threshold and all single-read MOTUs removed as a conservative contamination/false-positive control. Convert the read table to presence/absence MOTU detection per sample, group samples by location, and exclude locations that cannot support a curve (in the source study, Belize was excluded from the curve set). Compute sample-based species accumulation curves per location with vegan::specaccum, using 100 random permutations of sample order per location, and plot all location-specific curves together in a single figure (the paper's Fig. 4 layout). The key decision point is visual: a curve that has not plateaued indicates that sampling did not capture total diversity at that location; in this study no Caribbean or New Caledonian curve reached a plateau, with Caribbean curves flattening after ~10 samples except the Bahamas. Custom R scripts supporting this analysis are available from the metabarpark GitHub repository, and the underlying filtered sequences are deposited on Mendeley Data.

## Related tools

- **vegan** (R package used to compute sample-based species accumulation curves per location via specaccum with 100 random permutations of sample order)
- **R** (Language in which the MOTU table is converted to presence/absence, curves are computed, and the Fig. 4 figure is rendered; custom scripts from the metabarpark repository) — http://github.com/metabarpark
- **sumaclust** (Upstream MOTU delimitation step (99% similarity threshold) that produces the MOTU set feeding the accumulation curves)

## Examples

```
library(vegan); sac <- specaccum(motu_pa[loc == 'bahamas', ], method = 'random', permutations = 100); plot(sac)
```

## Evaluation signals

- species_accumulation_curves.png exists and matches the Fig. 4 reference layout (same number of location curves, comparable axes)
- Rendered curves visually reproduce the reported non-plateauing pattern for all Caribbean and New Caledonian locations
- Each location's curve is computed from exactly 100 random permutations of sample order (vegan::specaccum, method=random)
- Single-read MOTUs are absent from the input table and Belize is excluded from the curve set, matching the paper's conservative filtering
- Per-location curve counts match the number of samples per location after grouping the presence/absence matrix

## Limitations

- Absence of a plateau indicates under-sampling but does not estimate how many additional species would be found with more effort
- Curves depend on upstream filtering choices: single-read MOTUs were removed as a conservative control, so potential false positives from contamination or sequencing error are excluded along with potentially real detections
- The 127 bp COI fragment could not separate some closely related species (e.g. Carcharhinus, Rhizoprionodon, Negaprion MOTUs 100% identical to multiple BOLD species), so 'MOTU richness' may undercount true species richness
- Some species present at sampling sites were not detected at all (e.g. Ginglymostoma cirratum observed visually but no reads recovered) due to primer mismatches

## Evidence

- [discussion] MOTUs delimited at 99% similarity feed the richness tables: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] Conservative removal of single-read MOTUs before richness assessment: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] Custom R scripts for the analysis are publicly available: "All custom R scripts are publicly available from http://github.com/metabarpark."
- [abstract] Richness context: at least 21 shark species across the sampled regions: "We identified at least 21 shark species, from both Caribbean and Pacific Coral Sea water samples"
