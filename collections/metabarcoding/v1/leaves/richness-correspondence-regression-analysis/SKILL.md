---
name: richness-correspondence-regression-analysis
description: Use when a paired dataset of OTU/ASV tables and independent ground-truth
  richness surveys (e.g., plant field inventories) is available and the goal is to
  measure how curation or filtering changes OTU-richness vs species-richness correspondence
  (R²).
when_to_use_negative:
- No paired ground-truth inventory (e.g., field survey species counts) exists to regress
  OTU richness against, so R² correspondence cannot be computed
- The goal is taxonomic composition or β-diversity rather than α-diversity correspondence;
  the paper notes current tail-discarding approaches 'may be adequate for assessing
  composition and turnover, but much less so for α-diversity'
- The analysis targets identification of individual rare species — the article warns
  'LULU curation may thus not be suitable for identifying single rare species in community
  data sets'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: LULU
  role: Post-clustering curation of OTU tables before recomputing richness-vs-plant-richness
    R²; run with minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the initial OTU clustering methods whose tables feed the singleton-removal
    vs LULU-curation R² comparison; also usable to produce the LULU match list
- name: SWARM
  role: Initial OTU clustering method whose curated vs uncurated tables are compared
    in the richness-correspondence regression
- name: CROP
  role: Initial OTU clustering method included in the R² comparison across clustering
    approaches
- name: DADA2
  role: Initial OTU definition method included in the R² comparison; recommended as
    the initial OTU definition step in the article's discussion
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:c122574d46d884dfd23f020c034b0ecebf34bf95dbfebf11fa6b75f21195198d
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 18
---

# richness-correspondence-regression-analysis

## Summary

Quantifies how well molecular OTU richness tracks independent field-survey species richness by regressing per-site OTU richness against per-site plant richness (R²) on the 130-site Danish plant ITS2 soil dataset, and uses the R² gain to benchmark singleton removal against LULU post-clustering curation.

## When to use

Use when a paired dataset of OTU/ASV tables and independent ground-truth richness surveys (e.g., plant field inventories) is available and the goal is to measure how curation or filtering changes OTU-richness vs species-richness correspondence (R²).

## When NOT to use

- No paired ground-truth inventory (e.g., field survey species counts) exists to regress OTU richness against, so R² correspondence cannot be computed
- The goal is taxonomic composition or β-diversity rather than α-diversity correspondence; the paper notes current tail-discarding approaches 'may be adequate for assessing composition and turnover, but much less so for α-diversity'
- The analysis targets identification of individual rare species — the article warns 'LULU curation may thus not be suitable for identifying single rare species in community data sets'

## Inputs

- Initial OTU tables (samples as columns, OTUs as rows, unique OTU ids as row names) produced with VSEARCH, SWARM, CROP and DADA2
- Paired per-site vascular plant field-survey richness values for the 130 Danish sites
- OTU representative sequences and match list (e.g., from BLASTn or VSEARCH, minimum match 84%) for LULU curation

## Outputs

- Per-site OTU richness vectors from uncurated, singleton-removed, and LULU-curated tables
- R² values from OTU-richness vs plant-richness regressions per clustering method and treatment
- comparison_table.csv tabulating R² improvements for singleton removal versus LULU curation across VSEARCH, SWARM, CROP and DADA2

## How to apply

Load the initial OTU tables produced with VSEARCH, SWARM, CROP and DADA2 for the 130 Danish sites together with the paired vascular plant field-survey richness data (564 species total; per-site richness 6–93). Filter singletons (OTUs with total read count of one) from each table, compute per-site OTU richness, and regress it against per-site plant richness to obtain baseline and post-filter R² values. Recompute the same regression on LULU-curated tables (minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95). Tabulate the R² improvements across clustering methods (comparison_table.csv); the expected result is a mean R² improvement of ~0.03 for singleton removal versus ~0.27 for LULU curation, with the strongest gains for the greedy methods VSEARCH and SWARM.

## Related tools

- **LULU** (Post-clustering curation of OTU tables before recomputing richness-vs-plant-richness R²; run with minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the initial OTU clustering methods whose tables feed the singleton-removal vs LULU-curation R² comparison; also usable to produce the LULU match list)
- **SWARM** (Initial OTU clustering method whose curated vs uncurated tables are compared in the richness-correspondence regression)
- **CROP** (Initial OTU clustering method included in the R² comparison across clustering approaches)
- **DADA2** (Initial OTU definition method included in the R² comparison; recommended as the initial OTU definition step in the article's discussion)

## Examples

```
curated_result <- lulu(otutable, matchlist, minimum_ratio_type='min', minimum_ratio=1, minimum_match=84, minimum_relative_cooccurence=0.95); summary(lm(otu_richness ~ plant_richness))$r.squared
```

## Evaluation signals

- Mean R² improvement over the uncurated tables should be approximately 0.03 for singleton removal and approximately 0.27 for LULU curation
- The singleton-removed tables should retain only OTUs with total read count greater than one; LULU-curated tables should have fewer OTUs than uncurated tables while preserving total read counts via merging with parent OTUs
- Per-site plant richness used as the response should fall within the reported range of 6 to 93 species across the 130 sites
- comparison_table.csv should contain R² improvement entries for each clustering method (VSEARCH, SWARM, CROP, DADA2), with the largest LULU-driven gains expected for VSEARCH and SWARM
- Each R² comes from a regression of per-site OTU richness on per-site plant richness across all 130 samples

## Limitations

- Singleton removal yields only marginal R² gains (~0.03) compared with LULU curation (~0.27), so it is a weak alternative to post-clustering curation
- The 'cost of curation' is a small loss of 'real' (rare, low-abundance) species, and LULU curation may not be suitable for identifying single rare species
- Validation was performed only on vascular plants with ITS2 soil metabarcoding from 130 Danish sites; relevance to other organism groups is inferred but not directly tested
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy and β-diversity measures), which can depress its R² contribution

## Evidence

- [results] Singleton removal offered only marginal gains compared to LULU curation: the mean improvement in the coefficient of determination (R2) for OTU richness vs plant richness was 0.03 for singleton removal versus 0.27 for LULU curation.: [evidence span withheld — anchor status: not_found]
- [intro] we used a plant data set for ITS2 (nuclear ribosomal internal transcribed spacer region 2) obtained from an extensive soil sampling across 130 field sites in Denmark: "we used a plant data set for ITS2 (nuclear ribosomal internal transcribed spacer region 2) obtained from an extensive soil sampling across 130 field sites in Denmark"
- [intro] A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93: "with field site species richness per site ranging from 6 to 93"
- [discussion] The improvement by LULU curation was most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs: "The improvement by LULU curation was most pronounced for the two greedy methods, VSEARCH and SWARM, which initially had the highest number of OTUs"
- [readme] The `lulu` function will return a curated OTU table and some statistics and information on the curation in the object `curated_result`.: "The `lulu` function will return a curated OTU table and some statistics and information on the curation in the object `curated_result`."
