---
name: 16s-region-wise-sample-grouping
description: Use when a genus-level ASV table from 16S rRNA amplicon sequencing must
  be stratified by amplified hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) before
  comparing alpha diversity, beta diversity, or taxonomic resolving power between
  regions.
when_to_use_negative:
- Input samples were all sequenced over the same hypervariable region (no region stratification
  exists to compare)
- The goal is taxonomic assignment or denoising (Deblur/DADA2) rather than between-region
  comparison of already-assigned ASVs
- Samples lack metadata identifying which combined hypervariable region was amplified
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_3697
tools:
- name: phyloseq
  role: R package used to hold the genus-level ASV table and subset samples per hypervariable-region
    group before diversity estimation
- name: vegan
  role: R package used alongside phyloseq to compute alpha diversity indices on the
    region-grouped samples
- name: Deblur
  role: Produces the ASV-level abundance data that is collapsed at genus level and
    then grouped by region
- name: Greengenes
  role: Reference database for the genus-level taxonomic collapse applied consistently
    across all region groups
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:8b0ff69ef29ad7968788e43fc01317d1ce271ba8a33f7ea992830c633ff2b011
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Greengenes
  - R
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# 16s-region-wise-sample-grouping

## Summary

Groups genus-level 16S rRNA ASV abundance samples by the combined hypervariable region amplified (V1–V2, V3–V4, V5–V7, V7–V9) so downstream diversity and resolving-power comparisons are made between region-defined cohorts rather than pooled samples.

## When to use

Use when a genus-level ASV table from 16S rRNA amplicon sequencing must be stratified by amplified hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9) before comparing alpha diversity, beta diversity, or taxonomic resolving power between regions.

## When NOT to use

- Input samples were all sequenced over the same hypervariable region (no region stratification exists to compare)
- The goal is taxonomic assignment or denoising (Deblur/DADA2) rather than between-region comparison of already-assigned ASVs
- Samples lack metadata identifying which combined hypervariable region was amplified

## Inputs

- Genus-level ASV abundance table (Deblur-derived, Greengenes-collapsed) from 33 sputum samples and ZymoBIOMICS mock community controls
- Per-sample metadata mapping each sample to its amplified combined hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- Four region-stratified sample subsets (V1–V2, V3–V4, V5–V7, V7–V9) of the phyloseq object
- Group-labeled abundance matrix ready for Kruskal–Wallis alpha diversity testing and pairwise region comparisons

## How to apply

Load the genus-level ASV abundance table from the study repository (github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification) and attach a metadata column recording which combined hypervariable region was amplified for each of the 33 sputum samples (plus ZymoBIOMICS mock community controls). Split the phyloseq object into four groups — V1–V2, V3–V4, V5–V7, V7–V9 — keeping the Greengenes-collapsed genus-level taxonomy consistent across groups. These region-wise groups then feed per-sample alpha diversity estimation (Shannon, inverse Simpson, Chao1 in R 3.4.1 with phyloseq/vegan), Kruskal–Wallis testing between region groups, and pairwise sensitivity/specificity comparisons. The grouping rationale is that region choice drives observed diversity: V7–V9 yields significantly lower alpha diversity than the other three combinations.

## Related tools

- **phyloseq** (R package used to hold the genus-level ASV table and subset samples per hypervariable-region group before diversity estimation)
- **vegan** (R package used alongside phyloseq to compute alpha diversity indices on the region-grouped samples)
- **Deblur** (Produces the ASV-level abundance data that is collapsed at genus level and then grouped by region)
- **Greengenes** (Reference database for the genus-level taxonomic collapse applied consistently across all region groups)

## Examples

```
Rscript -e 'ps <- readRDS("asv_genus_phyloseq.rds"); for (r in c("V1-V2","V3-V4","V5-V7","V7-V9")) assign(r, subset_samples(ps, region==r))'
```

## Evaluation signals

- Every sample appears in exactly one of the four region groups (V1–V2, V3–V4, V5–V7, V7–V9) with no unassigned samples
- Taxonomy is collapsed at genus level identically across all four groups (same Greengenes reference) so groups are comparable
- Downstream Kruskal–Wallis p-values on grouped data reproduce article values (Shannon p = 4.6E-07, Chao1 p = 7.1E-06, inverse Simpson p = 1E-07) within tolerance
- The V7–V9 group ranks lowest on all three alpha diversity indices relative to the other region groups
- Group sizes match the reported cohort (33 human sputum samples plus ZymoBIOMICS controls)

## Limitations

- All 33 sputum samples were profiled on Illumina MiSeq with its 300 bp length limit, which constrains profiling of longer region combinations such as V7–V9
- Taxonomy was assigned with Greengenes rather than more updated SILVA versions, which may bias genus-level composition differently per region
- The ZymoBIOMICS mock community contains only 10 species, affecting sensitivity and specificity of taxonomic classification when validating region comparisons
- Most other studies lack direct comparability because richness or entropy are niche dependent

## Evidence

- [full_text] Group samples by the hypervariable region amplified (V1–V2, V3–V4, V5–V7, V7–V9): [evidence span withheld — anchor status: not_found]
- [abstract] Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9: "Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [methods] Alpha diversity indices and rarefaction curves were estimated in R (version 3.4.1): "The alpha diversity indices and rarefaction curves were estimated in R (version 3.4.1)"
- [abstract] We compare the resolving powers of V1–V2, V3–V4, V5–V7, and V7–V9 to improve microbiome analyses in sputum samples: "We compare the resolving powers of V1–V2, V3–V4, V5–V7, and V7–V9 to improve microbiome analyses in sputum samples"
