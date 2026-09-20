---
name: amplicon-sequence-demultiplexing
description: Use when starting from raw pooled amplicon sequencing data (per-study
  demultiplexed raw sequence files from an EMP-style FTP deposit) and per-sample FASTQ
  tables are needed before OTU picking or Deblur processing.
when_to_use_negative:
- Input is already a per-sample feature/BIOM table — demultiplexing operates on pooled
  FASTQ reads and cannot be applied downstream of OTU picking
- Sequences are shotgun metagenomic or metabolomic data from the EMP500 multi-omics
  release, where amplicon demultiplexing does not apply
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: QIIME1
  role: 'Initial QIIME processing: split reads into per-sample FASTQ files by barcode
    and apply quality filtering; also used (with QIIME2) to merge the resulting BIOM
    tables and mapping files'
- name: QIIME2
  role: Merging per-study BIOM tables and mapping files into a single QC-filtered
    table
- name: Deblur
  role: Exact-sequence de novo OTU picking run on the demultiplexed reads, trimmed
    to 90 bp to match the V4 tag length
  repo: https://github.com/cuttlefishh/deblur
- name: Python
  role: Scripting and validation of the merged table (studies, samples, unique sequence
    counts); supplementary metadata tables use Python/Pandas
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:31f6803969d158e646beda253ac4851ad899ed1de8b4e74edbcdcdee7fd37fc9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Python
  - QIIME1
  - QIIME2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# amplicon-sequence-demultiplexing

## Summary

Demultiplexing splits pooled Illumina amplicon reads into per-sample FASTQ files using barcode sequences, the entry point of the EMP 16S Release 1 processing pipeline that feeds per-study 'demux' tables into Deblur for exact-sequence inference of 90-bp V4 tag sequences.

## When to use

Use when starting from raw pooled amplicon sequencing data (per-study demultiplexed raw sequence files from an EMP-style FTP deposit) and per-sample FASTQ tables are needed before OTU picking or Deblur processing.

## When NOT to use

- Input is already a per-sample feature/BIOM table — demultiplexing operates on pooled FASTQ reads and cannot be applied downstream of OTU picking
- Sequences are shotgun metagenomic or metabolomic data from the EMP500 multi-omics release, where amplicon demultiplexing does not apply

## Inputs

- per-study demultiplexed raw sequence files (FASTQ) from the EMP 16S Release 1 FTP deposit (ftp://ftp.microbio.me/emp/release1)
- barcode/index sequences used to split pooled reads into per-sample FASTQ files

## Outputs

- per-study demultiplexed ('demux') per-sample FASTQ tables
- downstream merged QC-filtered Deblur 90-bp feature table (emp_deblur_90bp_qc.biom) after mc2 filtering, >=25-read sample retention, and rarefaction to 5,000 observations per sample

## How to apply

Download per-study raw sequence files and, as the initial QIIME processing step, split reads into per-sample FASTQ files by barcode to build per-study demultiplexed ('demux') tables; in this dataset 'demux' specifically describes the FASTQ data after splitting by barcode. The demultiplexed reads are then trimmed to 90 bp to match the V4 tag length (V4 region, starting after primer 515f) and passed to Deblur for exact-sequence de novo OTU picking. Downstream, apply the mc2 rule (minimum sequence count of 2 per OTU), retain only samples with >=25 total reads, and rarefy to 5,000 observations per sample before merging per-study BIOM tables with QIIME1 or QIIME2.

## Related tools

- **QIIME1** (Initial QIIME processing: split reads into per-sample FASTQ files by barcode and apply quality filtering; also used (with QIIME2) to merge the resulting BIOM tables and mapping files)
- **QIIME2** (Merging per-study BIOM tables and mapping files into a single QC-filtered table)
- **Deblur** (Exact-sequence de novo OTU picking run on the demultiplexed reads, trimmed to 90 bp to match the V4 tag length) — https://github.com/cuttlefishh/deblur
- **Python** (Scripting and validation of the merged table (studies, samples, unique sequence counts); supplementary metadata tables use Python/Pandas)

## Evaluation signals

- After the full pipeline (Deblur 90 bp, mc2 filter, >=25 reads, rarefaction to 5,000), the merged table should contain 307,572 unique sequences across 96 studies and 23,828 samples
- Demultiplexed FASTQ files must be split per sample by barcode with quality filtering applied before any OTU picking step
- Tag sequences must be 90-bp 16S rRNA gene sequences from the V4 region starting after primer 515f, as produced by Deblur
- Merged BIOM table should save as emp_deblur_90bp_qc.biom and be reproducible by merging per-study tables with QIIME1 or QIIME2

## Limitations

- The published release was QC-filtered with the mc2 rule and a >=25 total reads per sample threshold, so sample and sequence counts differ from unfiltered demultiplexed totals
- Analyses relied on exact sequences (Deblur) rather than clustered OTUs; mixing clustered and exact-sequence outputs breaks cross-study comparability
- Samples were rarefied to 5,000 observations per sample, so samples with fewer reads are excluded from rarefied analyses

## Evidence

- [methods] demultiplexing definition: "demux is shorthand for "demultiplexed", which describes the fastq data after it is split into per-sample fastq files using barcodes"
- [supplementary] Deblur tag sequences: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [supplementary] rarefaction threshold: "Samples were rarefied to 5,000 observations per sample."
- [readme] EMP Release 1 scope: "EMP 16S Release 1, a meta-analysis of the first 97 16S rRNA amplicon studies, has been published"
