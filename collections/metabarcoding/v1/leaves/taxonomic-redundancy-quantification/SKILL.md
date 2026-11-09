---
name: taxonomic-redundancy-quantification
description: Use when validating OTU tables (raw or LULU-curated, e.g. from VSEARCH,
  SWARM, CROP or DADA2) against ground-truth survey data, and you need to measure
  how many OTUs redundantly represent the same species.
when_to_use_negative:
- No reference-database assignments exist for the OTUs — the article's redundancy
  metric is defined via OTU-to-GenBank species assignment and cannot be computed without
  them
- The goal is identifying individual rare species rather than community-level biodiversity
  metrics — the authors note LULU curation may not be suitable for identifying single
  rare species
- Input is a taxon-free abundance matrix without OTU ids matching the centroid sequences,
  so OTUs cannot be linked to GenBank assignments
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: LULU
  role: Post-clustering curation whose effect on taxonomic redundancy is quantified
    (raw vs curated tables compared)
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Produces OTU-to-reference match lists / assignments used for OTU-to-GenBank
    species assignment underlying the redundancy metric
- name: VSEARCH
  role: One of the initial clustering methods whose raw and curated tables are compared
    for redundancy; also an alternative for producing match lists
- name: SWARM
  role: One of the initial clustering methods whose raw and curated tables are compared
    for redundancy
  repo: https://github.com/frederic-mahe/swarm
- name: CROP
  role: One of the initial clustering methods whose raw and curated tables are compared
    for redundancy
- name: DADA2
  role: One of the initial OTU-definition methods whose raw and curated tables are
    compared for redundancy
- name: GenBank
  role: Public reference database providing species assignments for the redundancy
    calculation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:ea827b1a02a1501e8fde2751c1df21f983123ce3158b25db93a8855e7b8dddb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - BLASTn
  - CROP
  - DADA2
  - GenBank
  - LULU
  - R
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# taxonomic-redundancy-quantification

## Summary

Quantifies taxonomic redundancy of an OTU table — the degree to which multiple OTUs map to the same species via reference-database (GenBank) assignments — as a validation metric for post-clustering curation with LULU. In the Frøslev et al. plant ITS2 study, redundancy dropped from 22–81% before curation to 6–13% at the 97% clustering level after LULU.

## When to use

Use when validating OTU tables (raw or LULU-curated, e.g. from VSEARCH, SWARM, CROP or DADA2) against ground-truth survey data, and you need to measure how many OTUs redundantly represent the same species.

## When NOT to use

- No reference-database assignments exist for the OTUs — the article's redundancy metric is defined via OTU-to-GenBank species assignment and cannot be computed without them
- The goal is identifying individual rare species rather than community-level biodiversity metrics — the authors note LULU curation may not be suitable for identifying single rare species
- Input is a taxon-free abundance matrix without OTU ids matching the centroid sequences, so OTUs cannot be linked to GenBank assignments

## Inputs

- OTU table (samples as columns, OTUs as rows, unique OTU ids as row names) — raw and LULU-curated versions for each method/level combination
- OTU-to-GenBank species assignments (from BLASTn/VSEARCH search of OTU centroid sequences against GenBank ITS2 references)
- Per-site vascular plant field survey richness for the 130 Danish sites (564 species total, 6–93 species per site)

## Outputs

- Taxonomic redundancy (fraction of OTUs mapping to the same species) per OTU table, raw vs curated
- Total OTU count per table (checked against the 564-species survey benchmark)
- Table 1-equivalent CSV (e.g. table1_validation_metrics.csv) with one row per method/level/curation-status including R2, slope, intercept, redundancy, total OTUs, average best GenBank match, and beta-diversity

## How to apply

Assign each OTU to species using public reference databases (GenBank, which held ITS2 data for 88% of the 564 plant species recorded in the Danish field survey), then compute the fraction of OTUs that share the same species assignment (taxonomic redundancy) per table. Compute this separately for each method/level combination (VSEARCH 98.5–95%, SWARM 99–95%, CROP 98–95%, DADA2 100%, DADA2+VSEARCH 98.5–95%) in both raw and LULU-curated form, alongside total OTU count, average best GenBank match similarity, regression metrics (R2, slope, intercept vs per-site survey richness), and beta-diversity compared to the survey value of 17.23. Correct application is judged by whether redundancy decreases after LULU curation: at the 97% level, curated tables should show 6–13% redundancy versus 22–81% before, with total OTUs reduced below the 564 surveyed plant species. Assemble one row per method/level/curation-status (e.g. a table1_validation_metrics.csv) for direct comparison across the 20 combinations.

## Related tools

- **LULU** (Post-clustering curation whose effect on taxonomic redundancy is quantified (raw vs curated tables compared)) — https://github.com/tobiasgf/lulu
- **BLASTn** (Produces OTU-to-reference match lists / assignments used for OTU-to-GenBank species assignment underlying the redundancy metric)
- **VSEARCH** (One of the initial clustering methods whose raw and curated tables are compared for redundancy; also an alternative for producing match lists)
- **SWARM** (One of the initial clustering methods whose raw and curated tables are compared for redundancy) — https://github.com/frederic-mahe/swarm
- **CROP** (One of the initial clustering methods whose raw and curated tables are compared for redundancy)
- **DADA2** (One of the initial OTU-definition methods whose raw and curated tables are compared for redundancy)
- **GenBank** (Public reference database providing species assignments for the redundancy calculation)

## Examples

```
summary(lm(otu_richness ~ survey_richness))$r.squared  # per method/level table, alongside redundancy = OTUs_per_species / total_OTUs
```

## Evaluation signals

- After LULU curation, taxonomic redundancy at the 97% clustering level should fall to 6–13%, down from 22–81% in the raw tables
- Post-curation total OTUs should fall below the 564 plant species recorded in the field survey
- Post-curation R2 (OTU-inferred vs survey site richness regression) should lie within the reported 0.56–0.71 range across the 20 method/level combinations
- Post-curation beta-diversity should approach the survey value of 17.23 (19.1–26.5 for all approaches except pure DADA2 at 36.3)
- Every method/level/curation-status combination should appear as one row in the assembled table (20 combinations, raw and curated)

## Limitations

- CROP OTU picking appeared suboptimal compared to the other methods investigated (low similarity to best GenBank match, poor redundancy and beta-diversity measures)
- GenBank contained ITS2 data for only 88% of the species recorded in the field survey, so redundancy is bounded by reference-database completeness
- LULU curation carries a small 'cost of curation': a loss of some real (rare, low-abundance) species

## Evidence

- [intro] GenBank species assignments underpin the metric: "we assigned OTUs to species in public reference databases (GenBank), which contained ITS2 data for 88% of species recorded in the field survey"
- [discussion] Redundancy is an explicit validation figure (Fig. 1c): "the similarity between OTU and best GenBank match (Fig. 1e) were far lower, and the taxonomic redundancy (Fig. 1c)"
- [intro] Ground-truth species count and per-site range: "A total of 564 plant species (approximately one third of the naturally occurring flora) were recorded in the study, with field site species richness per site ranging from 6 to 93"
- [abstract] Curation effect that the redundancy metric validates: "LULU curation consistently improves α-diversity estimates and other biodiversity metrics"
- [readme] Match-list format for the OTU sequence matching step: "The only requirements is that the match list has three columns with pair wise similarity scores for the OTUs."
