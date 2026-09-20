---
name: amplicon-read-demultiplexing
description: Use when raw pooled amplicon FASTQ reads are multiplexed and must be
  separated into per-sample FASTQ files by barcode before merging, quality filtering,
  and OTU/ASV feature-table construction.
when_to_use_negative:
- Input is already demultiplexed per-sample FASTQ files or an existing OTU/ASV feature
  table (e.g., otu_table.tsv)
- Input is shotgun metagenomic data, where the paper recommends the metagenomic analysis
  pipeline instead of the amplicon one
- Reads are already merged and quality filtered and the goal is only OTU/ASV inference
  or taxonomy assignment
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: QIIME 2
  role: Reproducible, interactive, community-supported pipeline platform implementing
    the demultiplexing, denoising/clustering (ASV/OTU), and feature-table steps of
    the recommended amplicon workflow
- name: Liu2020ProteinCell pipeline scripts
  role: Provide the paper's demultiplexing script and step-by-step reproducible amplicon
    analysis examples executed on the raw FASTQ reads
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:a6cabb5dabdbcec8e5d38e871e9b77109739703563343b59b3e3397d09929ebd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Liu2020ProteinCell pipeline scripts
  - QIIME 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# amplicon-read-demultiplexing

## Summary

Splits pooled (multiplexed) paired-end amplicon FASTQ reads into per-sample FASTQ files using sample barcode sequences, as the entry point of the Liu et al. 2020 recommended amplicon pipeline implemented in QIIME 2 and the YongxinLiu/Liu2020ProteinCell reproducible-analysis scripts.

## When to use

Use when raw pooled amplicon FASTQ reads are multiplexed and must be separated into per-sample FASTQ files by barcode before merging, quality filtering, and OTU/ASV feature-table construction.

## When NOT to use

- Input is already demultiplexed per-sample FASTQ files or an existing OTU/ASV feature table (e.g., otu_table.tsv)
- Input is shotgun metagenomic data, where the paper recommends the metagenomic analysis pipeline instead of the amplicon one
- Reads are already merged and quality filtered and the goal is only OTU/ASV inference or taxonomy assignment

## Inputs

- Pooled multiplexed paired-end amplicon FASTQ reads
- Sample barcode sequences / sample metadata mapping barcodes to samples
- Pipeline demultiplexing script from the YongxinLiu/Liu2020ProteinCell repository

## Outputs

- Per-sample demultiplexed FASTQ files ready for paired-end merging and quality filtering

## How to apply

Obtain the raw amplicon FASTQ reads and the pipeline scripts from the YongxinLiu/Liu2020ProteinCell repository accompanying the paper's reproducible guide. Run the pipeline's demultiplexing script on the pooled paired-end FASTQ using the sample barcode sequences from the sample metadata to write one FASTQ per sample. The per-sample outputs then feed directly into the next pipeline stages: merging paired-end reads and quality filtering to retain high-quality sequences for downstream inference. Downstream, the quality-filtered sequences are clustered into OTUs with UPARSE/USEARCH (e.g., 97% similarity) or denoised into exact ASVs with DADA2/Deblur, and QIIME 2 provides equivalent denoising/clustering functionality; a correct demultiplexing step is what guarantees the sample identities in the final quantified per-sample OTU/ASV count table.

## Related tools

- **QIIME 2** (Reproducible, interactive, community-supported pipeline platform implementing the demultiplexing, denoising/clustering (ASV/OTU), and feature-table steps of the recommended amplicon workflow)
- **Liu2020ProteinCell pipeline scripts** (Provide the paper's demultiplexing script and step-by-step reproducible amplicon analysis examples executed on the raw FASTQ reads)

## Evaluation signals

- Each output FASTQ contains only reads whose barcode matches a sample in the metadata; unassigned reads are reported and excluded from downstream stages
- Total read counts across per-sample FASTQ files reconcile with the input pooled FASTQ (accounting for unassigned reads)
- Downstream merge plus quality filtering retains high-quality sequences sufficient for the expected number of samples
- The final otu_table.tsv exists, contains per-sample sequence counts and taxonomy columns, and is consistent with the paper's step-by-step reproducible analysis guide

## Limitations

- Barcode sequencing errors or barcode collisions can misassign or drop reads during demultiplexing, propagating into the feature table
- The scripts are tied to the Liu2020ProteinCell reproducible-analysis workflow; barcode layout or primer design differing from the paper's may require adjusting the demultiplexing parameters

## Evidence

- [abstract] Recommended pipelines and tool selection: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [abstract] Step-by-step reproducible analysis: "Finally, a step-by-step reproducible analysis guide is introduced."
- [readme] Liu2020ProteinCell repository purpose: "Pipeline &amp; Reproducible analysis examples for amplicon and metagenome in microbiome"
