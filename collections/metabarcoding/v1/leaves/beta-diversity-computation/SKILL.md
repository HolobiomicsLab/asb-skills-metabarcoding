---
name: beta-diversity-computation
description: Use when evaluating whether OTU tables (e.g., from VSEARCH, SWARM, CROP,
  DADA2 at multiple clustering levels) have realistic sample-to-sample composition,
  by comparing their beta-diversity to an independent field-survey reference value.
when_to_use_negative:
- The goal is to identify or report on single rare species — the article notes LULU
  curation 'may thus not be suitable for identifying single rare species in community
  data sets'
- The data are not community abundance tables (OTU tables across multiple samples);
  beta-diversity comparison against survey ground truth is undefined without paired
  sample-level composition data
- Only alpha-diversity (per-site richness) is of interest — use the richness regression
  (R2, slope, intercept) workflow instead of beta-diversity
tools:
- name: LULU
  role: Post-clustering curation whose effect on beta-diversity is being quantified;
    produces the curated OTU tables evaluated against survey composition
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: One of the clustering methods producing the OTU tables (98.5–95% levels) whose
    beta-diversity is computed; can also produce the LULU match list via --usearch_global
- name: R
  role: Environment in which LULU is run and in which beta-diversity and other validation
    metrics (R2, slope, redundancy) are computed and assembled
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:e486a3ddb71b00888180905f511378f87f2abf229b120bb9e1b92d4cca0c3b6b
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# beta-diversity-computation

## Summary

Computes and compares beta-diversity (community composition turnover) for raw and LULU-curated OTU tables against ground-truth survey composition, as a validation metric for post-clustering curation of amplicon data.

## When to use

Use when evaluating whether OTU tables (e.g., from VSEARCH, SWARM, CROP, DADA2 at multiple clustering levels) have realistic sample-to-sample composition, by comparing their beta-diversity to an independent field-survey reference value.

## When NOT to use

- The goal is to identify or report on single rare species — the article notes LULU curation 'may thus not be suitable for identifying single rare species in community data sets'
- The data are not community abundance tables (OTU tables across multiple samples); beta-diversity comparison against survey ground truth is undefined without paired sample-level composition data
- Only alpha-diversity (per-site richness) is of interest — use the richness regression (R2, slope, intercept) workflow instead of beta-diversity

## Inputs

- Raw OTU table (samples as columns, OTUs as rows, unique OTU ids as row names) per clustering method/level
- LULU-curated OTU table (from lulu(otutable, matchlist) in R)
- Field-survey community composition per site (130 Danish sites, vascular plants)
- Match list of pairwise OTU similarities (e.g., BLASTn or VSEARCH output: query id, target id, percent similarity)

## Outputs

- Beta-diversity value per method/level/curation-status (raw vs curated)
- Comparison of each value against the survey-based beta-diversity reference (17.23)
- Rows in the assembled Table 1-equivalent validation metrics CSV

## How to apply

For each method/level combination (VSEARCH 98.5–95%, SWARM 99–95%, CROP 98–95%, DADA2 100%, DADA2+VS 98.5–95%), compute beta-diversity on the raw OTU table and again on the LULU-curated table produced by merging flagged erroneous 'daughter' OTUs into their 'parent' OTUs. Compare both against the beta-diversity derived from the paired per-site vascular plant survey data (130 Danish field sites; 564 species recorded, per-site richness 6–93). The survey value of 17.23 serves as the reference: after LULU curation, beta-diversity values of 19.1–26.5 are expected for all approaches except pure DADA2 (36.3). Assemble raw-vs-curated beta-diversity alongside R2, slope, intercept, total OTUs, taxonomic redundancy, and average best GenBank match into a single validation table (e.g., table1_validation_metrics.csv).

## Related tools

- **LULU** (Post-clustering curation whose effect on beta-diversity is being quantified; produces the curated OTU tables evaluated against survey composition) — https://github.com/tobiasgf/lulu
- **VSEARCH** (One of the clustering methods producing the OTU tables (98.5–95% levels) whose beta-diversity is computed; can also produce the LULU match list via --usearch_global)
- **R** (Environment in which LULU is run and in which beta-diversity and other validation metrics (R2, slope, redundancy) are computed and assembled)

## Evaluation signals

- Post-curation beta-diversity falls in the 19.1–26.5 range for all method/level combinations except pure DADA2 (36.3)
- Curated beta-diversity moves toward the survey-based reference value of 17.23 compared to the raw table
- Comparison is computed on all 20 method/level/curation-status combinations (raw and curated for each)
- Post-curation total OTUs fall below 564, the surveyed plant species count, corroborating realistic diversity estimates
- Each beta-diversity value is reported alongside R2, slope, intercept, taxonomic redundancy, and average best GenBank match in the assembled metrics table

## Limitations

- Pure DADA2 tables remained an outlier after curation (beta-diversity 36.3 vs 19.1–26.5 for other approaches)
- Curation costs a small loss of real (rare, low-abundance) species
- Validation was performed on vascular plant ITS2 soil data from 130 Danish sites; applicability to other organism groups is inferred but not directly tested

## Evidence

- [abstract] LULU curation consistently improves α-diversity estimates and other biodiversity metrics: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
- [abstract] OTU tables are produced with several different OTU definition algorithms and subsequently curated with LULU, and validated against field survey data: "OTU tables are produced with several different OTU definition algorithms and subsequently curated with LULU, and validated against field survey data"
- [results] and beta-diversity approached the survey value of 17.23 (19.1-26.5 for all approaches except pure DADA2 at 36.3): [evidence span withheld — anchor status: not_found]
- [readme] It has been shown that curation with **LULU** consistently result in more realistic diversity metrics.: "It has been shown that curation with **LULU** consistently result in more realistic diversity metrics."
- [intro] A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93: "A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93"
