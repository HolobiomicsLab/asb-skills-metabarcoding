---
name: biodiversity-database-cross-referencing
description: Use when a resolved eDNA metabarcoding species list needs independent
  validation against public occurrence records, e.g. checking whether taxa detected
  at a site (Vestamager) match national fauna databases.
when_to_use_negative:
- The taxon list has not yet been resolved to species level — unresolved OTUs cannot
  be meaningfully matched to faunistic records
- The sampling region lacks public occurrence database coverage (e.g. no Naturbasen/Fauna
  Europaea records), since absence of records may reflect database gaps rather than
  true absence
- The goal is community ecology comparison (e.g. clustering arthropod communities
  by plant species) rather than validation of detections against independent records
edam_operation: http://edamontology.org/operation_2422
tools:
- name: Naturbasen / allearter
  role: Danish national fauna database queried for occurrence records of each resolved
    taxon at Vestamager and Amager
- name: Fauna Europaea
  role: European faunal database used as a second source of occurrence records for
    cross-referencing taxa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:ef0fb1a87de1ca5f7a443e72024f87c46b89d07ddafe13b680609bf49158f1d6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fauna Europaea
  - Naturbasen / allearter
  edam_operation: http://edamontology.org/operation_2422
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# biodiversity-database-cross-referencing

## Summary

Validates eDNA-derived arthropod taxon lists against independent biodiversity databases (Naturbasen/allearter, Fauna Europaea) by flagging each resolved species as present or absent at a specific locality and a broader region, then tallying concordance counts and percentages.

## When to use

Use when a resolved eDNA metabarcoding species list needs independent validation against public occurrence records, e.g. checking whether taxa detected at a site (Vestamager) match national fauna databases.

## When NOT to use

- The taxon list has not yet been resolved to species level — unresolved OTUs cannot be meaningfully matched to faunistic records
- The sampling region lacks public occurrence database coverage (e.g. no Naturbasen/Fauna Europaea records), since absence of records may reflect database gaps rather than true absence
- The goal is community ecology comparison (e.g. clustering arthropod communities by plant species) rather than validation of detections against independent records

## Inputs

- Final resolved eDNA taxon list (e.g. 93 arthropod species from flower metabarcoding at Vestamager)
- Naturbasen / allearter Danish national fauna database occurrence records
- Fauna Europaea species occurrence records

## Outputs

- Per-taxon occurrence flag table (present/absent at Vestamager and on Amager) written to vestamager_occurrence_concordance.csv
- Summary concordance counts and percentages (e.g. 55/93 = 59% at Vestamager, 68/93 = 73% on Amager)

## How to apply

Load the final eDNA species list (e.g. the 93 Vestamager-resolved arthropod taxa from 16S rRNA/COI flower metabarcoding). Query each taxon against Naturbasen/allearter (Danish national fauna database) and Fauna Europaea to retrieve occurrence records. Flag each taxon as present or absent at the exact sampling locality and at the broader geographic unit (island of Amager), then tally counts and percentages (55/93 = 59% at Vestamager; 68/93 = 73% on Amager). Write per-taxon flags and summary counts to a CSV (vestamager_occurrence_concordance.csv). The rationale is that concordance between molecular detections and independent faunistic records supports the authenticity of eDNA recoveries; divergence signals mapping or database-coverage errors requiring review.

## Related tools

- **Naturbasen / allearter** (Danish national fauna database queried for occurrence records of each resolved taxon at Vestamager and Amager)
- **Fauna Europaea** (European faunal database used as a second source of occurrence records for cross-referencing taxa)

## Evaluation signals

- Computed concordance counts reproduce the reported statistics: 55/93 (59%) taxa at Vestamager and 68/93 (73%) on Amager
- The per-taxon output table contains a present/absent flag for every one of the 93 resolved taxa at both spatial scales (locality and island)
- vestamager_occordance CSV is complete: no missing rows, and flags sum to the reported tallies (55 + 38 = 93; 68 + 25 = 93)
- Divergence from the reported 59%/73% values triggers review for mapping or database-coverage errors rather than being silently accepted

## Limitations

- Database coverage is incomplete: taxa absent from Naturbasen/Fauna Europaea records may still be genuinely present, so concordance percentages reflect database coverage as well as detection accuracy
- Occurrence records are presence-only; lack of a record does not demonstrate true absence at the locality
- The article notes the eDNA experiment was not exhaustive — a higher arthropod richness could be obtained — so the cross-referenced list is a subset of the local fauna

## Evidence

- [other] How well do the arthropod species recovered by eDNA metabarcoding of wild flowers at Vestamager correspond with independent faunistic occurrence records?: [evidence span withheld — anchor status: not_found]
- [other] concordance finding: [evidence span withheld — anchor status: not_found]
- [abstract] wild flowers as eDNA source: "Here, we test the potential of wild flowers as a novel source of arthropod eDNA."
- [abstract] primer targets: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
