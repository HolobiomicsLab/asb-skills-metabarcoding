---
name: arthropod-diversity-count-summarization
description: Use when comparing taxonomic recovery between two primer sets/markers
  in eDNA metabarcoding data, e.g. counting unique and shared arthropod families per
  gene from MOTU-to-family assignment tables and mock-community recovery rates.
when_to_use_negative:
- Input is raw sequencing reads or unclustered MOTU sequences rather than per-gene
  MOTU-to-family assignment tables — taxonomy must already be assigned per MOTU.
- The analysis goal is community clustering or composition comparison between plant
  species rather than per-marker diversity count summarization.
- The markers compared are not the mitochondrial 16S rRNA and COI primer sets described
  for the flower eDNA data.
tools:
- name: Bioinformatic-tools
  role: Repository hosting the bioinformatic workflow and scripts used to process
    the 16S and COI MOTU assignment tables for the family-count comparison
  repo: tobiasgf/Bioinformatic-tools
- name: pandas
  role: Python library used to load the per-gene MOTU-to-family assignment tables
    and compute set intersections of families per marker
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:23984b690756e84b77c9551c360803c90540598f6b64f4cd958e371773da4c36
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
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# arthropod-diversity-count-summarization

## Summary

Summarizes arthropod diversity recovered per metabarcoding marker (16S rRNA vs COI) from flower eDNA MOTU-to-family assignment tables, producing a comparison table of unique families, shared families, total families (67), and mock-sample recovery rates.

## When to use

Use when comparing taxonomic recovery between two primer sets/markers in eDNA metabarcoding data, e.g. counting unique and shared arthropod families per gene from MOTU-to-family assignment tables and mock-community recovery rates.

## When NOT to use

- Input is raw sequencing reads or unclustered MOTU sequences rather than per-gene MOTU-to-family assignment tables — taxonomy must already be assigned per MOTU.
- The analysis goal is community clustering or composition comparison between plant species rather than per-marker diversity count summarization.
- The markers compared are not the mitochondrial 16S rRNA and COI primer sets described for the flower eDNA data.

## Inputs

- per-gene MOTU-to-family assignment tables for the 16S rRNA marker (one row per MOTU with assigned family)
- per-gene MOTU-to-family assignment tables for the COI marker (one row per MOTU with assigned family)
- mock-community sample assignment tables (5 mock samples per marker)

## Outputs

- coi_16s_family_comparison.csv: comparison table with one row per summary statistic (unique 16S families, families unique to COI, families shared by both genes, total 67 families, mock recovery per marker such as 2/5 COI vs 4/5 16S)

## How to apply

Load the per-gene MOTU-to-family assignment tables for the 16S rRNA and COI markers with pandas, keeping one row per MOTU with its assigned family. Count unique arthropod families per marker separately and identify the 7 families unique to 16S. Compute the set intersection of families between the two markers to determine how many of the 67 total families were recovered by both genes. For the mock-community samples, tabulate per marker how many of the 5 mock samples yielded recoveries (2/5 for COI vs 4/5 for 16S). Assemble these counts into a single comparison table with one row per summary statistic and write it to coi_16s_family_comparison.csv; the rationale is that COI yielded higher arthropod diversity than 16S in the flower eDNA results.

## Related tools

- **Bioinformatic-tools** (Repository hosting the bioinformatic workflow and scripts used to process the 16S and COI MOTU assignment tables for the family-count comparison) — tobiasgf/Bioinformatic-tools
- **pandas** (Python library used to load the per-gene MOTU-to-family assignment tables and compute set intersections of families per marker)

## Examples

```
python -c "import pandas as pd; s=pd.read_csv('16S_families.csv'); c=pd.read_csv('COI_families.csv'); print(len(set(s.family)), len(set(s.family)&set(c.family))); pd.DataFrame({'stat':['unique_16S','shared','total']}).to_csv('coi_16s_family_comparison.csv')"
```

## Evaluation signals

- Number of unique arthropod families unique to 16S equals 7.
- Total families across both markers equals 67 (matching the article's reported 67 families).
- Mock-community recovery counts match 2/5 for COI and 4/5 for 16S.
- Output CSV coi_16s_family_comparison.csv has exactly one row per summary statistic and columns for 16S, COI, and shared counts.
- Set intersection of family sets between markers is non-empty and its size plus marker-unique counts sum to the total family count.

## Limitations

- The experiment was not exhaustive and higher arthropod richness could be obtained with this eDNA approach, so counts represent a lower bound on diversity.
- Comparison relies on family-level assignment of MOTUs per marker; differences in primer bias between 16S and COI confound direct diversity comparison (e.g. COI recovered more families, but 16S recovered more mock samples, 4/5 vs 2/5).

## Evidence

- [abstract] two primer sets metabarcoding: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] total families context: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders"
- [methods] per-marker family counting: [evidence span withheld — anchor status: not_found]
- [results] mock recovery counts: [evidence span withheld — anchor status: not_found]
