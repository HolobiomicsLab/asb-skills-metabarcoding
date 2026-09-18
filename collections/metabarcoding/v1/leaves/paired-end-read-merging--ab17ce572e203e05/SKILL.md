---
name: paired-end-read-merging
description: Use when raw paired-end amplicon FASTQ reads have been demultiplexed
  into per-sample files and must be merged and quality-filtered before OTU clustering
  or ASV denoising.
when_to_use_negative:
- Input reads are single-end, so there is no overlap to merge
- Data are shotgun metagenomic rather than amplicon reads; metagenomic analysis follows
  the paper's separate metagenomic pipeline
- Input is already a merged sequence set or an existing OTU/ASV feature table
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: Implements equivalent merging, quality-filtering, denoising (DADA2/Deblur),
    and clustering functionality for building OTU/ASV feature tables from merged paired-end
    amplicon reads
- name: YongxinLiu/Liu2020ProteinCell
  role: Provides the pipeline scripts and reproducible analysis examples for amplicon
    processing, including demultiplexing and merging steps
  repo: https://github.com/YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:a6730732c481287d5eba04c01bdecd1577550846a346c7874c28369f27fb39c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - QIIME 2
  - YongxinLiu/Liu2020ProteinCell
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# paired-end-read-merging

## Summary

Merge quality-filtered paired-end amplicon FASTQ reads into single consensus sequences as a prerequisite step for OTU/ASV inference in the amplicon analysis pipeline recommended by Liu et al. 2020.

## When to use

Use when raw paired-end amplicon FASTQ reads have been demultiplexed into per-sample files and must be merged and quality-filtered before OTU clustering or ASV denoising.

## When NOT to use

- Input reads are single-end, so there is no overlap to merge
- Data are shotgun metagenomic rather than amplicon reads; metagenomic analysis follows the paper's separate metagenomic pipeline
- Input is already a merged sequence set or an existing OTU/ASV feature table

## Inputs

- Demultiplexed per-sample paired-end amplicon FASTQ files (forward and reverse reads)

## Outputs

- Merged, quality-filtered per-sample sequence files suitable for OTU clustering or ASV denoising
- Downstream: otu_table.tsv feature table with per-sample sequence counts and taxonomy columns

## How to apply

Following the workflow surfaced from the Liu2020ProteinCell reproducible analysis guide, first demultiplex the pooled FASTQ reads into per-sample files using the sample barcodes, then merge the paired-end reads by their overlapping regions and apply quality filtering to retain only high-quality sequences. The merged, filtered sequences serve as input to feature inference — either clustering into OTUs with UPARSE/USEARCH at 97% similarity or denoising into exact ASVs with DADA2/Deblur as implemented in QIIME 2. Merging improves sequence accuracy because errors in the overlap region of the forward and reverse reads are reconciled, and it provides full-length amplicon coverage needed for reliable taxonomy assignment. Downstream, assign taxonomy against a reference database and quantify feature frequencies per sample to build the otu_table.tsv feature table.

## Related tools

- **QIIME 2** (Implements equivalent merging, quality-filtering, denoising (DADA2/Deblur), and clustering functionality for building OTU/ASV feature tables from merged paired-end amplicon reads)
- **YongxinLiu/Liu2020ProteinCell** (Provides the pipeline scripts and reproducible analysis examples for amplicon processing, including demultiplexing and merging steps) — https://github.com/YongxinLiu/Liu2020ProteinCell

## Evaluation signals

- Confirm that merged reads show a valid overlap between forward and reverse read pairs with no unmerged high-abundance pairs
- Confirm quality filtering retains high-quality sequences and discards reads failing the filter, as described in the reproducible guide
- Confirm downstream otu_table.tsv exists and contains per-sample sequence counts and taxonomy columns
- Cross-check feature-table results against the paper's step-by-step reproducible analysis guide for consistency

## Limitations

- Poor-quality reverse reads or short amplicon overlap regions can cause a large fraction of pairs to fail merging
- Merging applies to the amplicon pipeline only; the paper notes methods such as metatranscriptome, metaproteome, and metabolome analysis are not widely accepted due to high cost and complex experimental and analysis methods
- Choice between OTU clustering (97% UPARSE) and ASV denoising is a decision point that changes downstream outputs

## Evidence

- [abstract] Recommended pipelines for amplicon analysis: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [abstract] Reproducible analysis guide: "Finally, a step-by-step reproducible analysis guide is introduced."
- [readme] Repository purpose: "Pipeline &amp; Reproducible analysis examples for amplicon and metagenome in microbiome"
