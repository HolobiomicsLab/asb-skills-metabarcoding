---
name: biodiversity-metric-validation-against-field-survey
description: Use when you have an OTU/ASV table from environmental samples paired
  with independent ground-truth species inventories (e.g., field surveys) and need
  to test whether a curation or filtering step improves α-diversity correspondence.
when_to_use_negative:
- No paired ground-truth inventory exists for the sampled sites — the R2 correspondence
  cannot be computed and the validation is meaningless
- You only need taxonomic composition or β-diversity assessment rather than α-diversity
  validation; the article notes rank-abundance tail-discard approaches may be adequate
  for composition but less so for α-diversity
- The survey and molecular data are not site-paired or sample-matched, so per-site
  richness regression is not defensible
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3305
tools:
- name: LULU
  role: Produces the curated OTU tables whose richness is validated against field-survey
    data; curation effect is evaluated against plant survey ground truth
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the initial OTU-clustering methods producing tables to validate; also
    usable to build the match list for LULU
- name: SWARM
  role: Initial OTU-clustering method producing one of the OTU tables validated against
    plant survey richness
- name: CROP
  role: Initial OTU-clustering method producing one of the OTU tables validated against
    plant survey richness
- name: DADA2
  role: Initial OTU-clustering method producing one of the OTU tables validated against
    plant survey richness; recommended for initial OTU definition
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:d24415a2bd161159f921a1c365800974cfc28d3764aa6e1a0f142e63e242cb0e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - CROP
  - DADA2
  - LULU
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# biodiversity-metric-validation-against-field-survey

## Summary

Validate molecular biodiversity estimates by regressing per-site OTU richness from metabarcoding tables against paired field-survey species richness, using R2 correspondence as the metric. Grounded in the Frøslev et al. 2017 LULU validation on 130 Danish sites of plant ITS2 soil metabarcoding paired with vascular plant survey data.

## When to use

Use when you have an OTU/ASV table from environmental samples paired with independent ground-truth species inventories (e.g., field surveys) and need to test whether a curation or filtering step improves α-diversity correspondence.

## When NOT to use

- No paired ground-truth inventory exists for the sampled sites — the R2 correspondence cannot be computed and the validation is meaningless
- You only need taxonomic composition or β-diversity assessment rather than α-diversity validation; the article notes rank-abundance tail-discard approaches may be adequate for composition but less so for α-diversity
- The survey and molecular data are not site-paired or sample-matched, so per-site richness regression is not defensible

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids as row names) for each clustering method (VSEARCH, SWARM, CROP, DADA2) — uncurated and curated variants
- Paired field-survey species richness data per site (e.g., vascular plant counts for the 130 Danish sites)
- Optionally, singleton-removed OTU tables (OTUs with total read count of one filtered out)

## Outputs

- Per-site OTU richness values from each table variant
- R2 values from regressing OTU richness against field-survey species richness per table variant and clustering method
- Comparison table (e.g., comparison_table.csv) of R2 improvements for each curation/filtering strategy per clustering method

## How to apply

Load the OTU tables (produced with VSEARCH, SWARM, CROP or DADA2) and the paired per-site field-survey richness data; in the reference study these were plant ITS2 soil metabarcoding data from 130 Danish sites paired with botanical survey richness (per-site species richness 6–93). Compute per-site OTU richness (presence/absence per site) from each table variant — uncurated baseline, singleton-removed, and LULU-curated (lulu with minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95). Regress per-site OTU richness against per-site plant species richness and record the coefficient of determination (R2) for each variant. Compare R2 improvements across clustering methods to judge whether the curation step moved molecular richness toward ground truth; in the reference comparison singleton removal gave a mean R2 improvement of ~0.03 versus ~0.27 for LULU curation. Tabulate the per-method improvements (e.g., comparison_table.csv) so the correspondence gain is auditable per OTU definition algorithm.

## Related tools

- **LULU** (Produces the curated OTU tables whose richness is validated against field-survey data; curation effect is evaluated against plant survey ground truth) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the initial OTU-clustering methods producing tables to validate; also usable to build the match list for LULU)
- **SWARM** (Initial OTU-clustering method producing one of the OTU tables validated against plant survey richness)
- **CROP** (Initial OTU-clustering method producing one of the OTU tables validated against plant survey richness)
- **DADA2** (Initial OTU-clustering method producing one of the OTU tables validated against plant survey richness; recommended for initial OTU definition)

## Examples

```
curated_result <- lulu(otutable, matchlist, minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95); R2 <- summary(lm(otu_richness ~ plant_richness))$r.squared
```

## Evaluation signals

- Mean R2 improvement over uncurated tables should approximate 0.03 for singleton removal and approximately 0.27 for LULU curation in the reference setup
- LULU-curated tables must yield consistently higher OTU-richness vs plant-richness R2 than uncurated tables across all four clustering methods
- Per-site OTU richness vectors must have length equal to the number of sites (130 in the reference dataset) with no missing samples
- comparison_table.csv must contain one R2 improvement row per clustering method per strategy (singleton removal vs LULU)
- The LULU curation parameters must match the reference configuration (minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95) for comparability

## Limitations

- The 'cost of curation' is a small loss of real (rare, low-abundance) species; LULU curation may not be suitable for identifying single rare species in community data sets
- Validation was performed only for vascular plants; relevance to other organism groups is inferred but not directly tested
- The improvement by LULU curation is most pronounced for greedy methods (VSEARCH, SWARM) that initially had the highest OTU counts; gains may be smaller for other algorithms

## Evidence

- [abstract] LULU curation consistently improves α-diversity estimates and other biodiversity metrics: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
- [intro] Singleton removal marginal vs LULU: "Furthermore, we evaluated the effect of singleton removal in comparison to curation"
- [intro] All tables subsequently curated with the LULU algorithm and curation effect evaluated against plant survey ground truth: "All tables were subsequently curated with the LULU algorithm, and the curation effect was evaluated against ground truth in the form of plant survey data"
- [intro] A total of 564 plant species were recorded, per-site richness 6 to 93: "A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93"
- [readme] LULU requires an OTU table with samples as columns and OTUs as rows: "The requirements of **LULU** are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names."
- [discussion] Improvement by LULU most pronounced for greedy methods: "The improvement by LULU curation was most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs"
