---
name: structured-table-construction-from-reported-figures
description: Use when a paper reports counts and approximate percentages only in prose
  (e.g. '118 out of 233 eukaryotic OTUs', '~55% Fungi') and they must be consolidated
  into a structured CSV profile table for reuse.
when_to_use_negative:
- The input is already a feature/OTU table or CSV — no transcription from prose is
  needed
- The target quantities require recomputation from raw reads or abundance matrices
  rather than transcription of reported summary figures
- Only qualitative statements are available with no numeric counts or percentages
  to tabulate
tools:
- name: pandas
  role: Builds the (category, group, value) DataFrame from extracted figures and writes
    eukaryotic_otu_profile.csv
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:33ffd6b856bce920b5160bd47473de1e5bbaf30b1e651ef11b84fe7ed469057a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - pandas
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# structured-table-construction-from-reported-figures

## Summary

Transcribes figures reported in prose (OTU totals, kingdom and functional-group percentages) from a metabarcoding article into a machine-readable (category, group, value) CSV via pandas, preserving the paper's approximate ('~') qualifiers as validation ranges rather than exact values.

## When to use

Use when a paper reports counts and approximate percentages only in prose (e.g. '118 out of 233 eukaryotic OTUs', '~55% Fungi') and they must be consolidated into a structured CSV profile table for reuse.

## When NOT to use

- The input is already a feature/OTU table or CSV — no transcription from prose is needed
- The target quantities require recomputation from raw reads or abundance matrices rather than transcription of reported summary figures
- Only qualitative statements are available with no numeric counts or percentages to tabulate

## Inputs

- Article prose (abstract/results) reporting eukaryotic OTU totals and percentage breakdowns
- Reported figures: 233 eukaryotic OTUs, 118 known species; kingdom shares Fungi/Plantae/Animalia; fungal functional-group percentages

## Outputs

- eukaryotic_otu_profile.csv with columns (category, group, value) containing the OTU counts and percentage rows
- Parseable pandas DataFrame of the same schema

## How to apply

Read the article abstract and results to extract the reported counts verbatim: for this trout-farm ITS2 metabarcoding study, 233 total eukaryotic OTUs of which 118 were known species. Record the kingdom-level shares of the identified OTUs (~55% Fungi, ~33% Plantae, ~12% Animalia) and the fungal functional-group proportions among known species (~45% plant pathogens, ~28% mushrooms/yeasts, ~11% volatile/malodor producers, ~8% fish pathogens, ~8% human pathogens). Assemble the rows into a pandas DataFrame with exactly three columns — category, group, value — separating counts (233, 118) from percentage entries so each can be validated against its own tolerance. Write the table to eukaryotic_otu_profile.csv with df.to_csv(index=False), then re-parse it to confirm it is a valid CSV and that numeric fields fall within the rounded '~' ranges stated in the article.

## Related tools

- **pandas** (Builds the (category, group, value) DataFrame from extracted figures and writes eukaryotic_otu_profile.csv)

## Examples

```
import pandas as pd; df = pd.DataFrame(rows, columns=['category','group','value']); df.to_csv('eukaryotic_otu_profile.csv', index=False)
```

## Evaluation signals

- CSV parses with pd.read_csv and has exactly the columns category, group, value
- Numeric count fields reproduce the reported values 233 (total eukaryotic OTUs) and 118 (known species)
- Percentage entries fall within the rounded '~' ranges stated in the article (e.g. ~55% Fungi, ~33% Plantae, ~12% Animalia; ~45% plant pathogens, ~28% mushrooms/yeasts)
- Functional-group percentages remain interpretable as shares of the 118 identified species, not of all 233 OTUs
- No rows lost or duplicated on round-trip write/read of the CSV

## Limitations

- Reported values are prefixed with '~', so the table can only be validated against approximate ranges, not exact percentages
- Percentages describe the 118 identified known species; the remaining 115 of 233 eukaryotic OTUs are unclassified/unassigned and outside the table's scope
- Some figures (e.g. the 55/33/12 kingdom split and the 11/8/8 functional groups) appear in prose without a matching supplementary table, so no independent file exists to cross-check against

## Evidence

- [abstract] abstract: "118 out of 233 of the eukaryotic OTUs were known species. Of these, ~ 45% were plant pathogens, and ~ 28% were mushroom/yeasts"
- [discussion] discussion: "Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned"
