---
name: arthropod-species-occurrence-lookup
description: Use when you have a resolved list of eDNA-derived arthropod taxa from
  a Danish sampling site and need to check which of them are independently documented
  in faunistic occurrence records at the site or a broader region.
when_to_use_negative:
- Input is not a resolved species list — raw metabarcoding reads or OTUs that have
  not been assigned to arthropod taxa cannot be meaningfully matched to faunistic
  occurrence records.
- The sampling site and region have no coverage in the queried databases (Naturbasen/allearter
  or Fauna Europaea); absence there reflects database coverage, not biological discordance.
- The goal is validating community ecology patterns (e.g. clustering of arthropod
  communities by plant species), which this concordance check does not address.
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_0610
tools:
- name: Naturbasen / allearter
  role: Danish national fauna database queried for occurrence records of each eDNA
    taxon at Vestamager and on Amager
- name: Fauna Europaea
  role: European faunistic database cross-checked for species occurrence and coverage
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:8b973973134f4ec067d22fd1acccf59e807211e2721506f934e53836b47d589f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fauna Europaea
  - Naturbasen / allearter
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# arthropod-species-occurrence-lookup

## Summary

Validates an eDNA metabarcoding species list (e.g. the 93 arthropod taxa resolved from wild flowers at Vestamager) against independent Danish faunistic occurrence databases (Naturbasen/allearter and Fauna Europaea) to quantify how many taxa are independently recorded at the sampling locality and region.

## When to use

Use when you have a resolved list of eDNA-derived arthropod taxa from a Danish sampling site and need to check which of them are independently documented in faunistic occurrence records at the site or a broader region.

## When NOT to use

- Input is not a resolved species list — raw metabarcoding reads or OTUs that have not been assigned to arthropod taxa cannot be meaningfully matched to faunistic occurrence records.
- The sampling site and region have no coverage in the queried databases (Naturbasen/allearter or Fauna Europaea); absence there reflects database coverage, not biological discordance.
- The goal is validating community ecology patterns (e.g. clustering of arthropod communities by plant species), which this concordance check does not address.

## Inputs

- Final eDNA species list of resolved arthropod taxa (e.g. the 93 taxa resolved from Vestamager wild-flower eDNA metabarcoding)
- Occurrence record databases: Naturbasen / allearter (Danish national fauna database) and Fauna Europaea

## Outputs

- Per-taxon occurrence flags (present/absent at Vestamager and on Amager)
- Summary concordance counts and percentages (expected benchmark: 55/93 = 59% at Vestamager, 68/93 = 73% on Amager)
- vestamager_occurrence_concordance.csv

## How to apply

Load the final eDNA species list (in the Vestamager study, 93 resolved arthropod taxa recovered from wild-flower eDNA metabarcoding using 16S rRNA and COI primers). For each taxon, query occurrence records in Naturbasen/allearter (the Danish national fauna database) and Fauna Europaea. Flag each taxon as present or absent at the specific sampling locality (Vestamager) and at the broader geographic unit (the island of Amager). Tally concordance counts and percentages — the reported benchmark is 55/93 taxa (59%) recorded at Vestamager and 68/93 (73%) on Amager. Write per-taxon occurrence flags and summary counts to vestamager_occurrence_concordance.csv.

## Related tools

- **Naturbasen / allearter** (Danish national fauna database queried for occurrence records of each eDNA taxon at Vestamager and on Amager)
- **Fauna Europaea** (European faunistic database cross-checked for species occurrence and coverage)

## Evaluation signals

- The computed counts must reproduce the reported statistic: 55/93 (59%) taxa recorded at Vestamager and 68/93 (73%) on Amager.
- Divergence from the reported 55/93 or 68/93 values indicates a mapping or database-coverage error requiring review.
- Every one of the 93 taxa carries both a Vestamager flag and an Amager flag (no missing flags).
- Percentages are computed against the full resolved taxon list (denominator = 93), not a filtered subset.
- vestamager_occurrence_concordance.csv contains per-taxon flags plus the summary counts.

## Limitations

- Database coverage in Naturbasen/allearter and Fauna Europaea may be incomplete, so absence of a record does not prove the species was absent from the site.
- Concordance is a sanity check on eDNA detections, not exhaustive — the article notes the flower eDNA experiment was not exhaustive and higher arthropod richness could be obtained.
- Only taxa resolved to species level (93 of at least 135 recovered arthropod species) can be checked; unresolved or higher-rank detections cannot be flagged.

## Evidence

- [other] Of the 93 Vestamager-resolved eDNA taxa, 55 (59%) have been recorded at the same sampling locality (Vestamager) and 68 (73%) on the island of Amager: [evidence span withheld — anchor status: not_found]
- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach.: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
