---
name: alpha-diversity-correlation-against-field-survey
description: Use when an OTU table (or curated vs raw versions) must be validated
  against independent per-site species richness/composition data, e.g. metabarcoding
  paired with botanical survey counts.
when_to_use_negative:
- No paired independent inventory data exist for the sampled sites, so no ground-truth
  richness can be regressed against OTU richness
- The goal is identifying single rare species rather than biodiversity estimates —
  LULU curation and this validation are not suited to that
- Input is a mock community with known genetic content; the authors note mock communities
  never have the complexity of real soil samples and recommend separate validation
  approaches
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: LULU
  role: Produces the curated OTU tables whose alpha-diversity and beta-diversity are
    regressed against field-survey ground truth
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the clustering methods (98.5-95%) whose OTU tables are validated; also
    used for post-DADA2 clustering and to produce the match list
- name: SWARM
  role: Clustering method (99-95%) whose OTU tables are included in the validation
    comparison
- name: CROP
  role: Clustering method (98-95%) whose OTU tables are validated against survey data
- name: DADA2
  role: Initial OTU-definition method (100% and 98.5-95% variants) whose tables are
    validated against survey data
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:dc5f2a8bdae7184373effedc96671b0d57d8b54f0ce773fe2b78013cc1b0e0bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - CROP
  - DADA2
  - LULU
  - R
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# alpha-diversity-correlation-against-field-survey

## Summary

Validates OTU tables against paired field-survey ground truth by regressing OTU-inferred site richness on per-site survey species richness (extracting R2, slope, intercept) and comparing beta-diversity, as done with 130 Danish soil sites and 564 surveyed plant species in the LULU paper.

## When to use

Use when an OTU table (or curated vs raw versions) must be validated against independent per-site species richness/composition data, e.g. metabarcoding paired with botanical survey counts.

## When NOT to use

- No paired independent inventory data exist for the sampled sites, so no ground-truth richness can be regressed against OTU richness
- The goal is identifying single rare species rather than biodiversity estimates — LULU curation and this validation are not suited to that
- Input is a mock community with known genetic content; the authors note mock communities never have the complexity of real soil samples and recommend separate validation approaches

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids as row names), raw and LULU-curated versions
- Per-site field-survey species richness for the same sites (e.g. vascular plant survey of 130 Danish sites)
- Optional: OTU-to-GenBank assignments for taxonomic redundancy and best-match similarity metrics

## Outputs

- Table of validation metrics (R2, slope, intercept, total OTUs, taxonomic redundancy, average best GenBank match, beta-diversity) per method/level/curation-status (e.g. table1_validation_metrics.csv)

## How to apply

Load the raw and curated OTU tables (samples as columns, OTUs as rows) together with per-site survey richness in R. For each table, regress OTU-inferred site richness on field-survey richness (e.g. survey values of 6-93 species per site, 564 species total) and extract R2, slope, and intercept. Compute beta-diversity for each table and compare against survey-based composition (survey value 17.23 in the paper). Optionally quantify taxonomic redundancy and average best GenBank match similarity per table. Assemble metrics one row per method/level/curation-status and judge success by post-curation metrics approaching survey ground truth (R2 improving 0.03-0.49, total OTUs dropping below 564, beta-diversity approaching 17.23).

## Related tools

- **LULU** (Produces the curated OTU tables whose alpha-diversity and beta-diversity are regressed against field-survey ground truth) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the clustering methods (98.5-95%) whose OTU tables are validated; also used for post-DADA2 clustering and to produce the match list)
- **SWARM** (Clustering method (99-95%) whose OTU tables are included in the validation comparison)
- **CROP** (Clustering method (98-95%) whose OTU tables are validated against survey data)
- **DADA2** (Initial OTU-definition method (100% and 98.5-95% variants) whose tables are validated against survey data)

## Examples

```
curated_result <- lulu(otutable_name, matchlist_name); curated_result$curated_table
```

## Evaluation signals

- Post-curation total OTUs fall below 564, the surveyed plant species count
- Post-curation R2 values fall within the reported 0.56-0.71 range across the 20 method/level combinations
- R2 improvement after curation is positive, in the 0.03-0.49 range (mean 0.27)
- Post-curation beta-diversity approaches the survey value 17.23 (expected 19.1-26.5 for all approaches except pure DADA2 at 36.3)
- Taxonomic redundancy at the 97% level drops from 22-81% before curation to 6-13% after

## Limitations

- The cost of curation is a small loss of real (rare, low-abundance) species
- LULU curation may not be suitable for identifying single rare species in community data sets
- Validation was performed only for vascular plants; relevance to other organism groups is inferred but not directly tested
- PCR/sequencing errors and intraspecific variation inflate sequence richness, and incomplete reference databases make it hard to separate errors from true OTUs

## Evidence

- [intro] To validate the LULU algorithm, we used a plant data set for ITS2 ... obtained from an extensive soil sampling across 130 field sites in Denmark: "To validate the LULU algorithm, we used a plant data set for ITS2 (nuclear ribosomal internal transcribed spacer region 2) obtained from an extensive soil sampling across 130 field sites in Denmark,"
- [intro] A total of 564 plant species were recorded in the study, with field site species richness per site ranging from 6 to 93: "A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93"
- [abstract] OTU tables are produced with several different OTU definition algorithms and subsequently curated with LULU, and validated against field survey data: "OTU tables are produced with several different OTU definition algorithms and subsequently curated with LULU, and validated against field survey data"
- [readme] The OTU table contains 2425 OTUs destributed over 130 sites/samples.: "The OTU table contains 2425 OTUs destributed over 130 sites/samples."
- [discussion] The LULU algorithm is a function written for R, accessible along with documentation on GitHub: "The LULU algorithm is a function written for R, accessible along with documentation on GitHub (https://github.com/tobiasgf/lulu)"
