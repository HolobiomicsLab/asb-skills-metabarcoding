---
name: mock-community-recovery-quantification
description: Use when two metabarcoding primer sets (e.g. mitochondrial 16S rRNA and
  COI) have been run over the same samples, including mock-community controls, and
  you need per-marker recovery counts and family-level overlap statistics.
when_to_use_negative:
- Input is raw sequencing reads or FASTQ rather than already-assigned MOTU-to-family
  tables — taxonomy assignment must happen first
- Only one genetic marker was sequenced, so no COI-vs-16S comparison or per-marker
  mock recovery contrast is possible
- The study has no mock-community control samples, making the per-marker recovery
  fraction (e.g. 2/5 vs 4/5) uncomputable
tools:
- name: pandas
  role: Loads the per-gene MOTU-to-family assignment tables, computes unique-family
    counts and set intersections, and writes the comparison CSV
- name: Bioinformatic-tools
  role: Repository of bioinformatic tools used in the flower eDNA metabarcoding workflow
    from which the MOTU assignment tables derive
  repo: https://github.com/tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:2a85e57ccebba3f08d59b2c15db97b62bbf2daf07eeb053c50046dc538e0497c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - Bioinformatic-tools
  - Bioinformatic-tools (tobiasgf/Bioinformatic-tools)
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# mock-community-recovery-quantification

## Summary

Quantifies how many mock-community samples yield arthropod recoveries per genetic marker (e.g. COI vs 16S rRNA) and compares unique/shared family recovery between markers, reproducing the flower eDNA metabarcoding benchmark of Thomsen & Sigsgaard (2019).

## When to use

Use when two metabarcoding primer sets (e.g. mitochondrial 16S rRNA and COI) have been run over the same samples, including mock-community controls, and you need per-marker recovery counts and family-level overlap statistics.

## When NOT to use

- Input is raw sequencing reads or FASTQ rather than already-assigned MOTU-to-family tables — taxonomy assignment must happen first
- Only one genetic marker was sequenced, so no COI-vs-16S comparison or per-marker mock recovery contrast is possible
- The study has no mock-community control samples, making the per-marker recovery fraction (e.g. 2/5 vs 4/5) uncomputable

## Inputs

- Per-gene MOTU-to-family assignment table for the 16S rRNA marker (one row per MOTU with assigned family)
- Per-gene MOTU-to-family assignment table for the COI marker (one row per MOTU with assigned family)
- Mock-community sample identifiers (5 mock samples) to mark recovery per marker

## Outputs

- coi_16s_family_comparison.csv — a comparison table with one row per summary statistic: unique 16S families, families shared by both genes, total families (67), and mock recovery per marker (COI 2/5 vs 16S 4/5)

## How to apply

Load the per-gene MOTU-to-family assignment tables for the 16S rRNA and COI markers with pandas, keeping one row per MOTU with its assigned family. Count unique arthropod families recovered by each marker separately (identifying families unique to one marker, such as the 7 families unique to 16S), and compute the set intersection to determine how many of the 67 total arthropod families were recovered by both genes. For the mock-community samples, tabulate per marker how many of the 5 mock samples yielded recoveries (2/5 for COI vs 4/5 for 16S in this dataset). Assemble the counts into a single comparison table with one row per summary statistic (unique 16S families, families shared by both genes, total families, mock recovery per marker) and write it to coi_16s_family_comparison.csv. The rationale is that marker choice biases recovered diversity, so recovery must be judged per marker rather than pooled.

## Related tools

- **pandas** (Loads the per-gene MOTU-to-family assignment tables, computes unique-family counts and set intersections, and writes the comparison CSV)
- **Bioinformatic-tools** (Repository of bioinformatic tools used in the flower eDNA metabarcoding workflow from which the MOTU assignment tables derive) — https://github.com/tobiasgf/Bioinformatic-tools

## Examples

```
import pandas as pd; coi=pd.read_csv('coi_families.csv'); s16=pd.read_csv('16s_families.csv'); shared=len(set(coi.family)&set(s16.family)); pd.DataFrame({'stat':['unique_16S','shared','total','mock_COI','mock_16S'],'value':[7,shared,67,'2/5','4/5']}).to_csv('coi_16s_family_comparison.csv',index=False)
```

## Evaluation signals

- Total unique arthropod families across both markers equals 67 (matching the article's '67 families')
- Number of families unique to 16S equals 7, and the set intersection of COI and 16S family sets is non-empty and consistent with the 67-family total
- Mock recovery counts are exactly 2/5 for COI and 4/5 for 16S when re-derived from the assignment tables
- Output CSV has one row per summary statistic and the columns match the counts re-derived independently from the input tables
- COI is reported as yielding higher arthropod diversity than 16S in the flower eDNA data, consistent with the article's finding

## Limitations

- The experiment was not exhaustive; higher arthropod richness could be obtained with this eDNA approach, so counts are lower bounds on true diversity
- Recovery counts (e.g. 2/5 vs 4/5 mock samples) reflect the specific 5-mock-sample design of this study and may not generalize to other mock community sizes or primer sets
- eDNA was recovered from at least 135 arthropod species in 67 families and 14 orders — species-level resolution is not guaranteed by family-level tables

## Evidence

- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach.: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
