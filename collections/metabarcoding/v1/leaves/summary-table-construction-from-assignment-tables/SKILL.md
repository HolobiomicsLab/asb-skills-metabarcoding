---
name: summary-table-construction-from-assignment-tables
description: Use when you have per-marker MOTU-to-family assignment tables (e.g. COI
  and 16S from flower eDNA metabarcoding) and need a summary comparing unique families,
  shared families, and mock-sample recoveries per marker.
when_to_use_negative:
- Input is already a merged/comparison table rather than raw per-gene MOTU-to-family
  assignment tables
- The goal is community composition or clustering analysis rather than marker-to-marker
  family recovery counts
- Markers other than the COI and 16S rRNA primer sets without re-deriving the shared/unique
  family logic for those markers
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3055
tools:
- name: Bioinformatic-tools
  role: Repository hosting the bioinformatic tooling behind the per-gene assignment
    tables that this skill summarizes
  repo: tobiasgf/Bioinformatic-tools
- name: pandas
  role: Python library used to load the MOTU-to-family assignment tables, compute
    family set intersections/counts, and write the comparison CSV
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:9a898b18c9e96e51431fcbf34cc8ebdf7a4761ee5af00013a92fb8f2ff85fbce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - Bioinformatic-tools
  - Bioinformatic-tools (tobiasgf/Bioinformatic-tools)
  - pandas
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3055
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# Summary table construction from assignment tables

## Summary

Builds a compact per-statistic comparison table from per-gene MOTU-to-family assignment tables, quantifying how arthropod family recovery differs between the COI and 16S rRNA primer sets used in the flower eDNA metabarcoding study.

## When to use

Use when you have per-marker MOTU-to-family assignment tables (e.g. COI and 16S from flower eDNA metabarcoding) and need a summary comparing unique families, shared families, and mock-sample recoveries per marker.

## When NOT to use

- Input is already a merged/comparison table rather than raw per-gene MOTU-to-family assignment tables
- The goal is community composition or clustering analysis rather than marker-to-marker family recovery counts
- Markers other than the COI and 16S rRNA primer sets without re-deriving the shared/unique family logic for those markers

## Inputs

- Per-gene MOTU-to-family assignment table for the COI marker (one row per MOTU with its assigned family)
- Per-gene MOTU-to-family assignment table for the 16S rRNA marker (one row per MOTU with its assigned family)
- Mock-community sample recovery records per marker (5 mock samples)

## Outputs

- coi_16s_family_comparison.csv: one row per summary statistic (unique 16S families, families shared by both genes, total families, mock recovery per marker)

## How to apply

Load the per-gene MOTU-to-family assignment tables for the 16S rRNA and COI markers with pandas, keeping one row per MOTU with its assigned family. Count unique arthropod families recovered per marker (in the source study, 7 families were unique to 16S), then compute the set intersection of family sets to determine how many of the 67 total families were recovered by both genes. For the 5 mock-community samples, tabulate per marker how many samples yielded recoveries (2/5 for COI vs 4/5 for 16S). Assemble one row per summary statistic (unique 16S families, families shared by both genes, total families, mock recovery per marker) and write the result to coi_16s_family_comparison.csv.

## Related tools

- **Bioinformatic-tools** (Repository hosting the bioinformatic tooling behind the per-gene assignment tables that this skill summarizes) — tobiasgf/Bioinformatic-tools
- **pandas** (Python library used to load the MOTU-to-family assignment tables, compute family set intersections/counts, and write the comparison CSV)

## Examples

```
import pandas as pd; cmp = pd.DataFrame({'statistic': ['unique_16S_families','families_shared_both','total_families','mock_recovery_COI','mock_recovery_16S']}); cmp.to_csv('coi_16s_family_comparison.csv', index=False)
```

## Evaluation signals

- Total arthropod families in the merged table equals 67, matching the article's reported 'at least 135 arthropod species in 67 families and 14 orders'
- Set intersection of COI and 16S family sets plus marker-unique counts reconciles exactly to the 67 total families
- Exactly 7 families are flagged as unique to 16S
- Mock-community recoveries are 2/5 samples for COI and 4/5 samples for 16S
- Output CSV contains exactly one row per summary statistic with the expected statistic names and counts

## Limitations

- Mock-community results show marker bias: COI yielded recoveries in fewer mock samples (2/5) than 16S (4/5), so per-marker counts are not directly commensurable without this caveat
- The underlying experiment was not exhaustive; higher arthropod richness could be obtained with this eDNA approach, so the 67-family total is a lower bound
- The repository README for tobiasgf/Bioinformatic-tools is empty in the provided context, so tool-level parameters cannot be verified from documentation

## Evidence

- [abstract] abstract: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
