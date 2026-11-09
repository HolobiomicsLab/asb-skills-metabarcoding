---
name: phylum-relative-abundance-tabulation
description: Use when 16S rRNA amplicon reads (e.g. V3-V4 region) have been OTU-clustered
  and taxonomically assigned and you need a phylum-level relative-abundance profile
  of the prokaryotic community.
when_to_use_negative:
- Input is eukaryotic ITS2 amplicon data — the article classifies those with the 'Classify
  Sequences' tool using the Warcup V2 training data set, not SILVAngs/BLASTn phylum
  profiling
- Input is already a curated feature/OTU abundance table and the goal is alpha- or
  beta-diversity (e.g. Simpson index, Bray-Curtis), which this skill does not compute
- The goal is strain- or genus-level identification of specific taxa (e.g. malodor-producing
  genera such as Myxobacteria/Sorangium) rather than phylum-level composition
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0621
tools:
- name: SILVAngs
  role: Online automatic pipeline that performs the OTU clustering and taxonomic classification
    of compressed 16S FASTQ files against the SILVA rDNA database at default settings
- name: SINA v1.2.10
  role: SILVA Incremental Aligner used to align each read against the SILVA SSU rRNA
    reference within SILVAngs
- name: cd-hit-est v3.1.2
  role: Clusters reads into OTUs at identity criteria of 1.00 and 0.98, ignoring overhangs,
    prior to taxonomic assignment
- name: BLASTn
  role: Local search used to assign OTU taxonomy against the non-redundant SILVA SSU
    ref dataset (release no. 132)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:a1084380173760d36b6608d19be5f5136559cb3b21a7e19e187190d0c5a8eb1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - SILVAngs
  - SINA
  - SINA v1.2.10
  - cd-hit-est
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# phylum-relative-abundance-tabulation

## Summary

Tabulates phylum-level relative abundances and summary statistics (total species, total phyla, bacterial OTU fraction) from 16S rRNA metabarcoding data taxonomically classified via the SILVAngs pipeline, producing a structured composition table for prokaryotic communities.

## When to use

Use when 16S rRNA amplicon reads (e.g. V3-V4 region) have been OTU-clustered and taxonomically assigned and you need a phylum-level relative-abundance profile of the prokaryotic community.

## When NOT to use

- Input is eukaryotic ITS2 amplicon data — the article classifies those with the 'Classify Sequences' tool using the Warcup V2 training data set, not SILVAngs/BLASTn phylum profiling
- Input is already a curated feature/OTU abundance table and the goal is alpha- or beta-diversity (e.g. Simpson index, Bray-Curtis), which this skill does not compute
- The goal is strain- or genus-level identification of specific taxa (e.g. malodor-producing genera such as Myxobacteria/Sorangium) rather than phylum-level composition

## Inputs

- Compressed FASTQ files of merged 16S rRNA (V3-V4) amplicon reads from water samples
- SILVA SSU rRNA reference database (SILVAngs, default settings)
- Non-redundant SILVA SSU ref dataset, release no. 132 (for BLASTn assignment)

## Outputs

- Phylum-level relative abundance table (e.g. prokaryotic_composition.csv with columns phylum and relative_abundance_percent)
- Summary statistics: total bacterial species (1318), total phyla detected (17), and fraction of prokaryotic OTUs assigned to Bacteria (>99%)

## How to apply

Run classified reads through the SILVAngs pipeline: align each read against the SILVA SSU rRNA database with SINA v1.2.10, dereplicate identical reads and cluster OTUs using cd-hit-est v3.1.2 with identity criteria of 1.00 and 0.98 while ignoring overhangs, then assign taxonomy via local BLASTn against the non-redundant SILVA SSU ref dataset (release no. 132). Discard short reads and reads with more than 2% ambiguities or homopolymers before classification; leave reads whose '(% sequence identity + % alignment coverage)/2' does not exceed 93 as unclassified/unassigned. From the resulting assignments, extract phylum-level relative abundances (e.g. Proteobacteria, Bacteroidetes, Actinobacteria, Verrucomicrobia, Cyanobacteria) plus the total species count, total phyla count, and the fraction of prokaryotic OTUs assigned to Bacteria. Write these to a structured CSV (columns: phylum, relative_abundance_percent, plus summary fields) so downstream validation and comparison are reproducible.

## Related tools

- **SILVAngs** (Online automatic pipeline that performs the OTU clustering and taxonomic classification of compressed 16S FASTQ files against the SILVA rDNA database at default settings)
- **SINA v1.2.10** (SILVA Incremental Aligner used to align each read against the SILVA SSU rRNA reference within SILVAngs)
- **cd-hit-est v3.1.2** (Clusters reads into OTUs at identity criteria of 1.00 and 0.98, ignoring overhangs, prior to taxonomic assignment)
- **BLASTn** (Local search used to assign OTU taxonomy against the non-redundant SILVA SSU ref dataset (release no. 132))

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('prokaryotic_composition.csv'); assert abs(df[df.phylum=='Proteobacteria'].relative_abundance_percent.iloc[0]-75.57)<0.01"
```

## Evaluation signals

- CSV parses cleanly with required fields: phylum and relative_abundance_percent columns plus summary fields for total species, total phyla, and bacterial OTU fraction
- Proteobacteria relative abundance rounds to ~75.57% and Bacteroidetes to ~14.4% of prokaryotic OTUs
- More than 99% of detected prokaryotic OTUs are classified as Bacteria
- Total counts match the reported figures: 1318 bacterial species across 17 phyla
- Minor phyla (Actinobacteria 0.94%, Verrucomicrobia 0.62%, Cyanobacteria 0.25%) appear in the table with values matching reported abundances

## Limitations

- Reads with no or weak BLAST hits where the function '(% sequence identity + % alignment coverage)/2' did not exceed 93 remain unclassified or unassigned, so phylum profiles are based only on classifiable OTUs
- Exact farm locations were withheld at farmers' demand, limiting geographic reproducibility of the composition profile
- The profile reflects inflow- and pond-water samples from semi-intensive trout farms; it does not characterize feed microbial composition, which the article notes requires WGS-metagenomics follow-up

## Evidence

- [discussion] SILVAngs classifies OTUs via SILVA database alignment and BLASTn: "SILVAngs was used for data analysis using the compressed fastq files through an online automatic software pipeline based on the SILVA rDNA database"
- [discussion] OTU clustering parameters with cd-hit-est: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2"
- [discussion] BLASTn assignment against SILVA SSU ref release 132: "local BLASTn search against the non-redundant version of SILVA SSU ref dataset (release no. 132)"
- [abstract] Reported phylum-level abundances and bacterial OTU fraction: "more than 99% of the detected prokaryotic OTUs (Operational Taxonomic Unit identification) were bacteria as of ~ 75.57% were Proteobacteria, and ~ 14.4% were Bacteroidetes"
- [discussion] Quality filter before classification: "Short reads and/or with more than 2% ambiguities or homopolymers were discarded"
