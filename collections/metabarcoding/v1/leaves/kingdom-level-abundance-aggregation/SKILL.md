---
name: kingdom-level-abundance-aggregation
description: Use when an OTU-by-taxonomy table from ITS2 (or similar) metabarcoding
  must be summarized into kingdom-level percentages (e.g., ~55% Fungi, ~33% Plantae,
  ~12% Animalia) and functional-group proportions, then exported as a reportable profile
  table.
when_to_use_negative:
- Input is raw FASTQ or unassembled reads — OTU clustering (cd-hit-est v3.1.2) and
  taxonomic classification must precede this aggregation.
- The target is the prokaryotic 16S community profile (~75.57% Proteobacteria, ~14.4%
  Bacteroidetes), which is a separate phylum-level aggregation.
- The OTU table is already aggregated to kingdom level and only needs visualization
  or statistical testing.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0102
tools:
- name: pandas
  role: Assembles the (category, group, value) profile table and writes eukaryotic_otu_profile.csv
- name: BLASTn
  role: Provides the taxonomic assignment of eukaryotic OTUs against the SILVA SSU
    ref dataset (release no. 132) that feeds the kingdom tally
- name: cd-hit-est v3.1.2
  role: Upstream OTU clustering whose output eukaryotic OTU set (233 OTUs) is the
    aggregation input
- name: Classify Sequences (Warcup V2 training data set)
  role: ITS classification at 90% similarity with a minimum overlap of 75 bp used
    to identify the 118 known-species OTUs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:ebe609507dccef0582e215cf5f9819dd9174713aec809085a808fedf29db7558
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Classify Sequences (Warcup V2 training data set)
  - cd-hit-est v3.1.2
  - pandas
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# kingdom-level-abundance-aggregation

## Summary

Aggregates OTU-level taxonomic identifications from ITS2 metabarcoding into kingdom-level (Fungi/Plantae/Animalia) and fungal functional-group percentage profiles, exporting them as a structured CSV (e.g., eukaryotic_otu_profile.csv with 233 total OTUs, 118 known species).

## When to use

Use when an OTU-by-taxonomy table from ITS2 (or similar) metabarcoding must be summarized into kingdom-level percentages (e.g., ~55% Fungi, ~33% Plantae, ~12% Animalia) and functional-group proportions, then exported as a reportable profile table.

## When NOT to use

- Input is raw FASTQ or unassembled reads — OTU clustering (cd-hit-est v3.1.2) and taxonomic classification must precede this aggregation.
- The target is the prokaryotic 16S community profile (~75.57% Proteobacteria, ~14.4% Bacteroidetes), which is a separate phylum-level aggregation.
- The OTU table is already aggregated to kingdom level and only needs visualization or statistical testing.

## Inputs

- Eukaryotic OTU table from ITS2 metabarcoding of inflow- and pond-water samples (233 OTUs across three semi-intensive trout farms)
- Per-OTU taxonomic assignments from BLASTn against the non-redundant SILVA SSU ref dataset (release no. 132) and the 'Classify Sequences' tool (Warcup V2 training data set)
- Known-species identification list (118 of 233 eukaryotic OTUs)
- Functional-group annotation for identified fungal OTUs (plant pathogen, mushroom/yeast, volatile malodor producer, fish pathogen, human pathogen)

## Outputs

- eukaryotic_otu_profile.csv — a parseable profile table with columns (category, group, value) holding OTU counts (233 total, 118 known species) and kingdom/functional-group percentages

## How to apply

Start from the ITS2 metabarcoding OTU set (here: 233 eukaryotic OTUs from inflow- and pond-water of three semi-intensive trout farms) with per-OTU taxonomy obtained via BLASTn against the non-redundant SILVA SSU ref dataset (release no. 132) and the 'Classify Sequences' tool using the Warcup V2 training data set (90% similarity, minimum overlap 75 bp). Restrict the aggregation to the 118 OTUs identified as known species, then tally membership per kingdom (~55% Fungi, ~33% Plantae, ~12% Animalia) and per fungal functional group (~45% plant pathogens, ~28% mushrooms/yeasts, ~11% volatile malodor producers, ~8% fish pathogens, ~8% human pathogens). Leave OTUs with weak BLAST hits — where (% sequence identity + % alignment coverage)/2 does not exceed 93 — unclassified rather than forcing an assignment. Assemble the counts and percentages into a pandas DataFrame with columns (category, group, value) and write it to eukaryotic_otu_profile.csv. Validate by reloading the CSV and checking that the numeric fields reproduce the reported values (233 total eukaryotic OTUs, 118 known species) and that percentage entries fall within the rounded '~' ranges stated in the abstract.

## Related tools

- **pandas** (Assembles the (category, group, value) profile table and writes eukaryotic_otu_profile.csv)
- **BLASTn** (Provides the taxonomic assignment of eukaryotic OTUs against the SILVA SSU ref dataset (release no. 132) that feeds the kingdom tally)
- **cd-hit-est v3.1.2** (Upstream OTU clustering whose output eukaryotic OTU set (233 OTUs) is the aggregation input)
- **Classify Sequences (Warcup V2 training data set)** (ITS classification at 90% similarity with a minimum overlap of 75 bp used to identify the 118 known-species OTUs)

## Examples

```
profile = pd.DataFrame(rows, columns=["category","group","value"]); profile.to_csv("eukaryotic_otu_profile.csv", index=False)
```

## Evaluation signals

- eukaryotic_otu_profile.csv exists and reloads cleanly with pd.read_csv
- Numeric fields reproduce the reported counts: 233 total eukaryotic OTUs and 118 known species
- Kingdom percentages fall within the rounded ranges: ~55% Fungi, ~33% Plantae, ~12% Animalia
- Fungal functional-group proportions sum to ~100% (45 + 28 + 11 + 8 + 8)
- Table schema matches the specified columns: category, group, value

## Limitations

- Percentages are rounded approximations ('~') from the abstract; exact per-kingdom OTU counts are not reported.
- Only the 118 known-species OTUs are profiled; the remaining 115 eukaryotic OTUs are excluded from percentage calculations.
- Some OTUs remained unclassified or unassigned due to weak BLAST hits, where (% sequence identity + % alignment coverage)/2 did not exceed 93, reducing the identifiable fraction.

## Evidence

- [abstract] 118 out of 233 of the eukaryotic OTUs were known species. Of these, ~ 45% were plant pathogens, and ~ 28% were mushroom/yeasts: "118 out of 233 of the eukaryotic OTUs were known species. Of these, ~ 45% were plant pathogens, and ~ 28% were mushroom/yeasts"
- [discussion] Generated consensus sequences were subject to the "Classify Sequences" tool using the Warcup V2 training data set: "Generated consensus sequences were subject to the "Classify Sequences" tool using the Warcup V2 training data set"
- [discussion] Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned: "Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned"
