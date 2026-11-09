---
name: inflow-versus-pond-water-group-comparison
description: Use when you have a species/taxon-level OTU abundance table with sample
  metadata distinguishing inflow water from pond water, and need to identify taxa
  whose relative abundances differ significantly between the two water-source groups.
when_to_use_negative:
- Input is raw FASTQ reads or unclustered reads — OTU clustering and taxonomic classification
  must be done first (e.g., via the SILVAngs pipeline)
- The comparison of interest is between farm locations rather than water source —
  the paper found pond-water biodiversity is more related to location than water source,
  so a location-level comparison would need a different design (e.g., beta-diversity
  plus Mantel test)
- Sequencing data are eukaryotic ITS reads classified with the Warcup V2 training
  set — this skill addresses prokaryotic 16S taxa only
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: SILVAngs
  role: Online automatic pipeline producing the classified OTU abundance table (cd-hit-est
    clustering at identity 1.00/0.98, BLASTn against SILVA SSU ref release 132) that
    is the input to the group comparison
- name: XLSTAT 2017
  role: Statistical add-in used to run the two-sample t-tests on taxon relative abundances
    between inflow- and pond-water groups
- name: cd-hit-est v3.1.2
  role: Clustering of quality-filtered 16S reads into OTUs at identity criteria 1.00
    and 0.98 prior to abundance summarization
- name: BLASTn
  role: Local search assigning OTUs taxonomically against the non-redundant SILVA
    SSU ref dataset (release no. 132), defining the species-level taxa tested
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:3467c5f1226c21bac26400d3004c1db5f63de1add9c88800d209906323268dea
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - SILVAngs
  - XLSTAT 2017
  - cd-hit-est v3.1.2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# inflow-versus-pond-water-group-comparison

## Summary

Compares species-level relative abundances between inflow- (I-source) and pond-water (P-source) samples from trout farms using two-sample t-tests, to identify taxa significantly enriched in either water source.

## When to use

Use when you have a species/taxon-level OTU abundance table with sample metadata distinguishing inflow water from pond water, and need to identify taxa whose relative abundances differ significantly between the two water-source groups.

## When NOT to use

- Input is raw FASTQ reads or unclustered reads — OTU clustering and taxonomic classification must be done first (e.g., via the SILVAngs pipeline)
- The comparison of interest is between farm locations rather than water source — the paper found pond-water biodiversity is more related to location than water source, so a location-level comparison would need a different design (e.g., beta-diversity plus Mantel test)
- Sequencing data are eukaryotic ITS reads classified with the Warcup V2 training set — this skill addresses prokaryotic 16S taxa only

## Inputs

- SILVAngs-derived OTU relative-abundance table with taxonomic assignments (phylum, family, genus, species) from 16S V3-V4 amplicon data
- Sample metadata mapping each sample to inflow- (I-source) or pond-water (P-source) group across the three trout farm locations

## Outputs

- table1_significant_species_ttest.csv: CSV of significantly different species with phylum, family, genus, mean relative abundance and SEM per group, and t-test p-value
- List of taxa significantly enriched in I-source vs. P-source

## How to apply

Load the SILVAngs-derived OTU abundance table (16S reads clustered with cd-hit-est v3.1.2 at identity criteria 1.00 and 0.98 and classified by local BLASTn against the SILVA SSU ref release 132 dataset) together with sample metadata labelling each sample as I-source or P-source. Compute per-taxon relative abundances, then group samples by water source. For each species-level taxon, calculate the mean relative abundance and the standard error of the mean (SEM) separately for I-source and P-source groups. Run two-sample t-tests on the relative abundances between groups in XLSTAT 2017 (Addinsoft, Paris, France), as in the paper, and retain taxa with significant p-values (15 species in Table 1, spanning Bacteroidetes and Proteobacteria). Assemble the results into a table with phylum, family, genus, per-group mean abundance and SEM, and p-value. Interpretation in the paper: Bacteroidetes species (e.g., Marinoscillum, Dysgonomonas, Paludibacter) were up-represented in P-source, while some Proteobacteria (Simplicispira, Amaricoccus, Thioclava) were up-represented in I-source.

## Related tools

- **SILVAngs** (Online automatic pipeline producing the classified OTU abundance table (cd-hit-est clustering at identity 1.00/0.98, BLASTn against SILVA SSU ref release 132) that is the input to the group comparison)
- **XLSTAT 2017** (Statistical add-in used to run the two-sample t-tests on taxon relative abundances between inflow- and pond-water groups)
- **cd-hit-est v3.1.2** (Clustering of quality-filtered 16S reads into OTUs at identity criteria 1.00 and 0.98 prior to abundance summarization)
- **BLASTn** (Local search assigning OTUs taxonomically against the non-redundant SILVA SSU ref dataset (release no. 132), defining the species-level taxa tested)

## Examples

```
python scripts/significant_species_ttest.py --otu-table silvangs_otu_abundance.tsv --metadata samples_water_source.tsv --group-col water_source --groups I-source,P-source --output table1_significant_species_ttest.csv
```

## Evaluation signals

- Output CSV contains exactly 15 taxon rows with phylum, family, genus, mean relative abundance and SEM per group, and p-value all populated
- Significant taxa match the paper's Table 1: Bacteroidetes species (Marinoscillum, Dysgonomonas, Paludibacter, Mucilaginibacter, Pedobacter, Lacihabitans, Empedobacter) up in P-source; Simplicispira, Amaricoccus, Thioclava up in I-source; Alicycliphilus, Caenimonas, Orientia, Sphingomonadaceae bacterium up in P-source
- All significant taxa belong to only two phyla: Bacteroidetes and Proteobacteria, across 12 families
- Relative abundances are recomputed consistently (per-taxon, per-sample normalization) before group means and SEMs are calculated

## Limitations

- Taxa with weak BLAST hits — where (% sequence identity + % alignment coverage)/2 did not exceed 93 — remained unclassified or unassigned, so they cannot be tested at species level
- Statistical testing was performed in XLSTAT 2017; equivalent tools require reproducing the same t-test setup to match reported p-values
- Exact farm locations were withheld at farmers' demand, limiting geographic reproducibility of the comparison

## Evidence

- [results] t-test identified 15 significantly different species between I- and P-source: [evidence span withheld — anchor status: not_found]
- [discussion] SILVAngs pipeline produced the classified OTU table: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] OTU clustering parameters: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2"
- [discussion] Taxonomic classification threshold: "Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned"
- [discussion] Biodiversity driven by location, not water source: "the biodiversity and microbial enrichment of eukaryotes and prokaryotes in pond-water are more related to location than the water source"
- [discussion] Exact farm locations withheld: "The exact location of these farms cannot be disclosed based on the farmer demand."
