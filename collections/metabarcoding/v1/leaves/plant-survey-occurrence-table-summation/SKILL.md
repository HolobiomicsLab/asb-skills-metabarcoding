---
name: plant-survey-occurrence-table-summation
description: Use when an OTU table derived from plant ITS2 metabarcoding of soil (or
  similar) must be validated against paired field-survey species lists, by summarizing
  per-site taxonomic composition into species-validation categories before and after
  post-clustering curation.
when_to_use_negative:
- 'The goal is to identify individual rare, low-abundance species: LULU curation may
  not be suitable for identifying single rare species in community data sets'
- No paired field-inventory or reference data exist for the study region, so recaptured/unregistered
  species categories cannot be defined
- Input is a sequence file rather than an OTU table with samples as columns and unique
  OTU ids as row names; clustering (e.g. VSEARCH, SWARM, DADA2) must be done first
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0637
tools:
- name: LULU
  role: Post-clustering curation of OTU tables that merges co-occurring, sequence-similar
    daughter OTUs into more abundant parent OTUs; curated tables are then compared
    with survey lists
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: Initial clustering method producing OTU tables and/or internal match lists
    (--usearch_global --self --id .84 --iddef 1) that feed LULU curation
- name: SWARM
  role: Alternative initial OTU clustering method whose tables are curated and evaluated
    against survey ground truth
- name: CROP
  role: Alternative initial OTU clustering method whose tables are curated and evaluated
    against survey ground truth
- name: DADA2
  role: Alternative initial OTU definition method; combined with VSEARCH clustering
    and LULU as the recommended approach
- name: GenBank
  role: Public reference database (ITS2) used to assign OTUs to species and label
    imperfect reference matches
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:fe560f7354ba4e4a8cd273210f1ebf0e7afa305a3006a43cc49a14fabb594202
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - CROP
  - DADA2
  - GenBank
  - LULU
  - SWARM
  - VSEARCH
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# plant-survey-occurrence-table-summation

## Summary

Compares OTU-derived per-site plant species inventories (before and after LULU curation) against field-survey species lists by summing occurrence counts into per-site proportions of imperfect reference matches, recaptured, unregistered, redundant, and lost species, as in Table 2 of the 130-site Danish ITS2 validation.

## When to use

Use when an OTU table derived from plant ITS2 metabarcoding of soil (or similar) must be validated against paired field-survey species lists, by summarizing per-site taxonomic composition into species-validation categories before and after post-clustering curation.

## When NOT to use

- The goal is to identify individual rare, low-abundance species: LULU curation may not be suitable for identifying single rare species in community data sets
- No paired field-inventory or reference data exist for the study region, so recaptured/unregistered species categories cannot be defined
- Input is a sequence file rather than an OTU table with samples as columns and unique OTU ids as row names; clustering (e.g. VSEARCH, SWARM, DADA2) must be done first

## Inputs

- Curated and uncurated OTU tables (samples as columns, OTUs as rows, unique OTU ids as row names) from VSEARCH, SWARM, CROP and DADA2 clustering
- Match list of internal OTU similarities (query OTU id, target OTU id, percent similarity), e.g. from BLASTn or VSEARCH
- Per-site vascular plant field survey species lists for the 130 Danish sites
- GenBank ITS2 reference database for species assignment of OTUs

## Outputs

- table2_species_validation_metrics.csv with per-site proportions (averaged across sites) of imperfect reference matches, recaptured species, unregistered species, redundant species, and lost recaptured species for each clustering method/similarity level, before and after LULU curation
- Curated OTU table with erroneous OTUs merged into parent OTUs (read counts preserved)

## How to apply

Load the curated LULU OTU tables (produced with default settings: minimum_match = 84, minimum_ratio = 1 (min), minimum_relative_cooccurence = 0.95) for each initial clustering method (VSEARCH, SWARM, CROP, DADA2) and similarity level, together with the per-site vascular plant survey lists for the 130 Danish sites. Assign OTUs to species using GenBank ITS2 reference data, labeling OTUs whose best match is not a perfect, uniquely annotated hit as 'imperfect reference matches'. For each site and table, compare the OTU-derived species inventory with the survey list and count imperfect reference matches, recaptured species (perfect database match uniquely annotated to a surveyed species), unregistered species, redundant species, and lost recaptured species. Convert counts to per-site proportions and average across sites for each method/similarity combination, then write the metrics to table2_species_validation_metrics.csv. Rationale: curation merges erroneous 'daughter' OTUs into 'parent' OTUs, so recaptured-species proportions should rise while few real (rare) species are lost.

## Related tools

- **LULU** (Post-clustering curation of OTU tables that merges co-occurring, sequence-similar daughter OTUs into more abundant parent OTUs; curated tables are then compared with survey lists) — https://github.com/tobiasgf/lulu
- **VSEARCH** (Initial clustering method producing OTU tables and/or internal match lists (--usearch_global --self --id .84 --iddef 1) that feed LULU curation)
- **SWARM** (Alternative initial OTU clustering method whose tables are curated and evaluated against survey ground truth)
- **CROP** (Alternative initial OTU clustering method whose tables are curated and evaluated against survey ground truth)
- **DADA2** (Alternative initial OTU definition method; combined with VSEARCH clustering and LULU as the recommended approach)
- **GenBank** (Public reference database (ITS2) used to assign OTUs to species and label imperfect reference matches)

## Examples

```
curated_result <- lulu(otutable, matchlist); curated_table <- curated_result$curated_table
```

## Evaluation signals

- Mean recaptured-species proportion rises from ~0.24 before curation to ~0.44 after curation
- Mean lost recaptured species remains ~0.02, consistent with the small 'cost of curation' of real rare species
- Curated table preserves total read counts because errors are merged with their designated parent OTUs from bottom to top
- LULU default criteria are met: minimum_match = 84, minimum_ratio = 1 (min type), minimum_relative_cooccurence = 0.95
- OTU tables contain 2425 OTUs distributed over 130 sites/samples in the example/validation setting

## Limitations

- The cost of curation is a small loss of real (rare, low-abundance) species, so single rare species may be misidentified as errors
- CROP OTU picking appeared suboptimal compared to other methods (low similarity to best GenBank match, poor redundancy and beta-diversity measures)
- Validation was performed only for vascular plants; relevance to other organism groups is inferred but not directly tested, and complete reference databases and error-free sequencing do not yet exist

## Evidence

- [intro] The core mechanism is the identification and merging of 'daughter' OTUs with consistently co-occurring, sequence similar, but more abundant 'parent' OTUs across a multi-sample data set: "The core mechanism is the identification and merging of ‘daughter’ OTUs with consistently co-occurring, sequence similar, but more abundant ‘parent’ OTUs across a multi-sample data set, under the"
- [intro] For additional parts of the validation procedure, we assigned OTUs to species in public reference databases (GenBank): "For additional parts of the validation procedure, we assigned OTUs to species in public reference databases (GenBank)"
- [readme] The OTU table contains 2425 OTUs destributed over 130 sites/samples.: "The OTU table contains 2425 OTUs destributed over 130 sites/samples."
- [readme] The requirements of LULU are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names.: "The requirements of **LULU** are that the OTU table has samples as columns and OTUs as rows, and that it has unique OTU id's as row names."
- [discussion] LULU curation may thus not be suitable for identifying single rare species in community data sets: "LULU curation may thus not be suitable for identifying single rare species in community data sets"
