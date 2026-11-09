---
name: eukaryotic-otu-summary-extraction
description: Use when an ITS2 metabarcoding study reports eukaryotic OTU totals, known-species
  identifications, and kingdom/functional-group percentages that must be compiled
  into a structured, validated summary table.
when_to_use_negative:
- Input is already a computed feature/OTU table (e.g. a BIOM file or abundance matrix)
  rather than reported summary statistics to be extracted
- The study reports prokaryotic (16S) OTU summaries only, with no eukaryotic/ITS kingdom
  or functional-group breakdown
- The percentages in the source are exact (not '~' rounded), so the article's tolerance-based
  validation would not apply
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_2814
- http://edamontology.org/topic_0621
tools:
- name: pandas
  role: Assemble extracted counts and percentages into a structured profile table
    and write it as CSV
- name: cd-hit-est v3.1.2
  role: Clustered ITS reads into eukaryotic OTUs whose counts and classifications
    feed the summary profile
- name: BLASTn
  role: Local search against SILVA SSU ref dataset (release no. 132) underlying the
    known-species identifications summarized in the table
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:32c4cffcda613d2540085ce88cf9ebba8d159f1e8a7eb401ceb36c85e1d17772
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - cd-hit-est v3.1.2
  - pandas
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_2814
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# eukaryotic-otu-summary-extraction

## Summary

Extracts and structures reported eukaryotic (ITS2) OTU summary statistics — total OTUs, known-species counts, kingdom-level distribution (Fungi/Plantae/Animalia), and fungal functional-group proportions — into a machine-readable profile table (CSV) for reproducible reporting of metabarcoding results.

## When to use

Use when an ITS2 metabarcoding study reports eukaryotic OTU totals, known-species identifications, and kingdom/functional-group percentages that must be compiled into a structured, validated summary table.

## When NOT to use

- Input is already a computed feature/OTU table (e.g. a BIOM file or abundance matrix) rather than reported summary statistics to be extracted
- The study reports prokaryotic (16S) OTU summaries only, with no eukaryotic/ITS kingdom or functional-group breakdown
- The percentages in the source are exact (not '~' rounded), so the article's tolerance-based validation would not apply

## Inputs

- Article abstract/results text reporting eukaryotic OTU counts (233 total, 118 known species) and kingdom/functional-group percentages
- ITS2 consensus sequence classifications (Classify Sequences tool with Warcup V2 training data set)
- BLASTn assignments against SILVA SSU ref dataset release no. 132

## Outputs

- eukaryotic_otu_profile.csv — a parseable CSV table with columns (category, group, value) holding OTU counts and percentage entries

## How to apply

Extract the reported eukaryotic OTU totals from the article abstract (here: 233 total eukaryotic OTUs from ITS2 sequencing of inflow- and pond-water samples from three semi-intensive trout farms, of which 118 were identified as known species). Record the kingdom-level breakdown of the identified eukaryotic OTUs — ~55% Fungi, ~33% Plantae, ~12% Animalia — and the fungal functional-group proportions among known species: ~45% plant pathogens, ~28% mushrooms/yeasts, ~11% volatile (malodor) producers, ~8% fish pathogens, and ~8% human pathogens. Assemble counts and percentages into a structured profile table (columns: category, group, value) using pandas and write it to CSV (eukaryotic_otu_profile.csv). Validate that numeric fields reproduce the reported values (233 total OTUs, 118 known species) and that percentage entries fall within the rounded '~' ranges stated in the article. Note that identifications derive from consensus sequences classified against the Warcup V2 training set and BLASTn against the SILVA SSU ref dataset (release no. 132), with OTUs clustered by cd-hit-est v3.1.2.

## Related tools

- **pandas** (Assemble extracted counts and percentages into a structured profile table and write it as CSV)
- **cd-hit-est v3.1.2** (Clustered ITS reads into eukaryotic OTUs whose counts and classifications feed the summary profile)
- **BLASTn** (Local search against SILVA SSU ref dataset (release no. 132) underlying the known-species identifications summarized in the table)

## Examples

```
import pandas as pd; df = pd.DataFrame({'category':['total_otus','known_species','kingdom','kingdom','kingdom','fungal_group','fungal_group','fungal_group','fungal_group','fungal_group'],'group':['eukaryotic','eukaryotic','Fungi','Plantae','Animalia','plant pathogens','mushrooms/yeasts','volatile (malodor) producers','fish pathogens','human pathogens'],'value':[233,118,55,33,12,45,28,11,8,8]}); df.to_csv('eukaryotic_otu_profile.csv', index=False)
```

## Evaluation signals

- The output CSV exists and is parseable (e.g. pd.read_csv succeeds without errors)
- Numeric fields reproduce the reported values: 233 total eukaryotic OTUs and 118 known species
- Percentage entries fall within the rounded '~' ranges stated in the article (e.g. Fungi ~55%, Plantae ~33%, Animalia ~12%; plant pathogens ~45%, mushrooms/yeasts ~28%)
- Fungal functional-group percentages sum to approximately 100% (45+28+11+8+8)
- Table schema matches the specified columns: category, group, value

## Limitations

- The reported kingdom/functional-group percentages are approximate ('~' rounded), so exact recomputation from raw OTU tables may differ slightly
- Some OTUs remained unclassified or unassigned due to weak BLAST hits where the function '(% sequence identity + % alignment coverage)/2' did not exceed the value of 93, so known-species counts are a subset of total OTUs
- The article does not disclose exact farm locations (farmers' request), limiting reproducibility of the underlying sampling context

## Evidence

- [abstract] Of 233 eukaryotic OTUs, 118 were known species; ~45% were plant pathogens and ~28% were mushroom/yeasts: "118 out of 233 of the eukaryotic OTUs were known species. Of these, ~ 45% were plant pathogens, and ~ 28% were mushroom/yeasts"
- [discussion] Unclassified OTUs due to weak BLAST hits: "Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned"
- [discussion] ITS consensus classification with Warcup V2: "Generated consensus sequences were subject to the "Classify Sequences" tool using the Warcup V2 training data set"
- [discussion] OTU clustering identity criteria: "ignoring overhangs and applying identity criteria of 1.00 and 0.98, respectively"
