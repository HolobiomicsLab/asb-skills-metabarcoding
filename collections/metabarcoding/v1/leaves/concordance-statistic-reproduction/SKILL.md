---
name: concordance-statistic-reproduction
description: Use when an eDNA-derived species list (e.g. resolved arthropod taxa from
  flower metabarcoding at a known site) needs validation against independent occurrence
  records, and reported concordance percentages must be reproduced or verified.
when_to_use_negative:
- The input is raw sequencing data (FASTQ) or reads rather than a curated, taxonomically
  resolved species list — the skill presupposes taxa already resolved from the metabarcoding
  pipeline.
- The taxa of interest are from a locality outside the coverage of Naturbasen/allearter
  and Fauna Europaea, so absence of records cannot be distinguished from database-coverage
  gaps.
- The goal is community-structure analysis (e.g. clustering of arthropod communities
  by plant species), not validation of detection concordance against occurrence records.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3050
tools:
- name: Naturbasen / allearter
  role: Danish national fauna database queried for occurrence records of each eDNA
    taxon at Vestamager and on Amager
- name: Fauna Europaea
  role: Faunistic reference database used alongside Naturbasen to verify European
    distribution of the resolved taxa
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:e56c89d8cdc3593649e0e01c243b6a23afa614cad97aaa96128cc49248b6e9a9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Fauna Europaea
  - Naturbasen / allearter
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# concordance-statistic-reproduction

## Summary

Cross-references a resolved eDNA metabarcoding species list against independent public faunistic occurrence databases (Naturbasen/allearter and Fauna Europaea) and reproduces reported concordance counts/percentages at nested geographic scales (locality and island), providing an independent sanity check on metabarcoding detections.

## When to use

Use when an eDNA-derived species list (e.g. resolved arthropod taxa from flower metabarcoding at a known site) needs validation against independent occurrence records, and reported concordance percentages must be reproduced or verified.

## When NOT to use

- The input is raw sequencing data (FASTQ) or reads rather than a curated, taxonomically resolved species list — the skill presupposes taxa already resolved from the metabarcoding pipeline.
- The taxa of interest are from a locality outside the coverage of Naturbasen/allearter and Fauna Europaea, so absence of records cannot be distinguished from database-coverage gaps.
- The goal is community-structure analysis (e.g. clustering of arthropod communities by plant species), not validation of detection concordance against occurrence records.

## Inputs

- Final eDNA species list of resolved arthropod taxa (93 Vestamager-resolved taxa)
- Sampling locality definition (Vestamager locality and Amager island boundaries)
- Occurrence records from Naturbasen / allearter (Danish national fauna database)
- Occurrence/distribution records from Fauna Europaea

## Outputs

- Per-taxon occurrence flag table (present/absent at Vestamager and on Amager) as vestamager_occurrence_concordance.csv
- Summary concordance counts and percentages (55/93 = 59% at Vestamager; 68/93 = 73% on Amager)

## How to apply

Start from the final list of resolved taxa — in the Vestamager case, the 93 arthropod taxa recovered by eDNA metabarcoding of wild flowers using 16S rRNA and COI primers. Query each taxon name against Naturbasen/allearter (the Danish national fauna database) and Fauna Europaea, and flag it as present or absent at two nested geographic levels: the exact sampling locality (Vestamager) and the enclosing island (Amager). Tally the presence flags and compute percentages at each level; the reference statistics are 55/93 (59%) of taxa recorded at Vestamager and 68/93 (73%) on Amager. Write both the per-taxon occurrence flags and the summary counts to vestamager_occurrence_concordance.csv. Treat any divergence from the reported counts as a signal of a mapping error or database-coverage gap requiring review, since concordance with faunistic records is the evidence that eDNA detections correspond to real, independently documented occurrences.

## Related tools

- **Naturbasen / allearter** (Danish national fauna database queried for occurrence records of each eDNA taxon at Vestamager and on Amager)
- **Fauna Europaea** (Faunistic reference database used alongside Naturbasen to verify European distribution of the resolved taxa)

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('vestamager_occurrence_concordance.csv'); print(df.vestamager.mean(), df.amager.mean())"
```

## Evaluation signals

- The computed counts reproduce the reported statistics exactly: 55/93 (59%) taxa recorded at Vestamager and 68/93 (73%) on Amager.
- The output CSV contains one row per resolved taxon (93 rows), each with non-missing presence/absence flags at both geographic levels.
- Amager-level presence count is ≥ Vestamager-level count (73% ≥ 59%), consistent with the nested locality/island relationship.
- Every taxon absent from both levels is reviewed for name-matching or synonymy errors before being accepted as a genuine non-concordance.
- Divergence from reported percentages triggers explicit review of locality mapping and database coverage rather than silent acceptance.

## Limitations

- Database coverage of Naturbasen/allearter and Fauna Europaea is incomplete, so absence of a record does not prove the species was not present at the site.
- Locality mapping errors (assigning records to the wrong level of Vestamager vs. Amager) can distort concordance counts.
- The eDNA approach was not exhaustive — the study notes 'an even higher arthropod richness could be obtained using this eDNA approach' — so concordance is assessed on the resolved subset (93 taxa), not all detections.

## Evidence

- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] eDNA recovery breadth underlying the 93 resolved taxa: [evidence span withheld — anchor status: not_found]
