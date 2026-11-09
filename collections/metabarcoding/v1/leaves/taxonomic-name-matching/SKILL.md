---
name: taxonomic-name-matching
description: Use when you have a resolved species list from eDNA metabarcoding and
  need to verify plausibility by checking each taxon against independent occurrence
  records for the sampling region (e.g., locality or island-level faunistic data).
when_to_use_negative:
- The taxon list is unresolved to species level (the concordance check presumes named
  taxa that can be matched to database records)
- The sampling region lies outside the geographic coverage of Naturbasen and Fauna
  Europaea (Danish/European fauna databases)
- The input is raw sequencing reads rather than a curated, resolved species list
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Naturbasen / allearter
  role: Danish national fauna database queried for occurrence records of each eDNA
    taxon at Vestamager and on Amager
- name: Fauna Europaea
  role: European faunal database used as an independent source of occurrence records
    for cross-checking eDNA taxa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:20d76b43e88f65da7fdb20046a835bdd65946ca55aa727cc65af1bd641c3bc2e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fauna Europaea
  - Naturbasen / allearter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# taxonomic-name-matching

## Summary

Cross-references a resolved eDNA taxon list against independent faunistic occurrence databases (Naturbasen/allearter, Fauna Europaea) to validate that metabarcoded arthropod species correspond to documented field records at specific localities.

## When to use

Use when you have a resolved species list from eDNA metabarcoding and need to verify plausibility by checking each taxon against independent occurrence records for the sampling region (e.g., locality or island-level faunistic data).

## When NOT to use

- The taxon list is unresolved to species level (the concordance check presumes named taxa that can be matched to database records)
- The sampling region lies outside the geographic coverage of Naturbasen and Fauna Europaea (Danish/European fauna databases)
- The input is raw sequencing reads rather than a curated, resolved species list

## Inputs

- Final resolved eDNA species list (e.g., 93 arthropod taxa from 16S rRNA and COI metabarcoding of wild flowers)
- Occurrence records from Naturbasen / allearter (Danish national fauna database)
- Occurrence records from Fauna Europaea

## Outputs

- Per-taxon occurrence flags (present/absent at Vestamager and on Amager) with summary counts and percentages in vestamager_occurrence_concordance.csv

## How to apply

Start from the final species list (e.g., the 93 resolved arthropod taxa recovered from wild flowers at Vestamager via 16S rRNA and COI metabarcoding). Query occurrence records for each taxon against Naturbasen / allearter (the Danish national fauna database) and Fauna Europaea, flagging each taxon as present or absent at the exact sampling locality (Vestamager) and at the broader regional scale (island of Amager). Decision point: a mismatch between eDNA detection and faunistic records does not automatically mean a false positive — it may reflect database coverage gaps, so divergent taxa should be flagged for review rather than discarded. Tally concordance counts and percentages (55 taxa, 59% at Vestamager; 68 taxa, 73% on Amager) and write per-taxon occurrence flags plus summary counts to vestamager_occurrence_concordance.csv. The rationale is that concordance with independent faunistic records validates the ecological realism of eDNA detections, complementing the molecular evidence that terrestrial arthropods leave traces of DNA on flowers they interact with.

## Related tools

- **Naturbasen / allearter** (Danish national fauna database queried for occurrence records of each eDNA taxon at Vestamager and on Amager)
- **Fauna Europaea** (European faunal database used as an independent source of occurrence records for cross-checking eDNA taxa)

## Evaluation signals

- Computed counts reproduce the reported statistics of 55/93 (59%) taxa recorded at Vestamager and 68/93 (73%) on Amager
- Every one of the 93 resolved taxa carries both a Vestamager flag and an Amager flag (no missing assignments)
- Output CSV (vestamager_occurrence_concordance.csv) contains per-taxon occurrence flags plus the two summary concordance percentages
- Percentages arithmetically match their counts (55/93 = 59%; 68/93 = 73%)
- Any divergence from the reported 59%/73% values indicates a mapping or database-coverage error requiring review

## Limitations

- Divergence from reported concordance values may stem from mapping errors or database-coverage gaps in Naturbasen/Fauna Europaea rather than false eDNA detections, requiring manual review
- The upstream eDNA experiment was not exhaustive — absence of a taxon in the eDNA list (or in faunistic records) does not imply true absence, since 'an even higher arthropod richness could be obtained using this eDNA approach'
- Concordance is assessed only at two spatial scales (sampling locality and island); finer or broader validation is not supported by the reported workflow

## Evidence

- [results] Of the 93 Vestamager-resolved eDNA taxa, 55 (59%) have been recorded at the same sampling locality (Vestamager) and 68 (73%) on the island of Amager: [evidence span withheld — anchor status: not_found]
- [methods] Query occurrence records for each taxon against Naturbasen / allearter (Danish national fauna database) and Fauna Europaea.: [evidence span withheld — anchor status: not_found]
- [abstract] We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups"
