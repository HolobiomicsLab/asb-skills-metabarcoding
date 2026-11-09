---
name: kegg-orthology-functional-annotation
description: Use when raw metagenomic FASTQ reads must be converted into functional
  feature tables, specifically KEGG orthology (KO) abundance tables alongside taxonomic
  profiles, per the Liu2020ProteinCell reproducible pipeline.
when_to_use_negative:
- Input is already a taxonomic or functional feature table (the skill operates on
  raw metagenomic FASTQ reads)
- The goal is amplicon (16S) analysis rather than shotgun metagenomics — the review
  recommends separate pipelines for amplicon and metagenomic analyses
- The study targets metatranscriptome, metaproteome, or metabolome functional characterization,
  which the review notes are not widely accepted due to high cost and complex experimental/analysis
  methods
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: KneadData
  role: 'Quality control and host-read removal: chains Trimmomatic trimming with Bowtie2
    host alignment to output clean reads for downstream functional profiling'
- name: Trimmomatic
  role: Adapter/quality trimming of raw FASTQ reads as part of the KneadData cleaning
    step
- name: Bowtie2
  role: Aligns reads against the host reference to remove host-contaminating reads
    during KneadData cleaning
- name: HUMAnN2
  role: Produces functional profiles from cleaned reads and regresses gene families
    to KEGG orthology (KO) abundance tables
- name: MetaPhlAn2
  role: Generates taxonomic abundance profiles from cleaned non-host reads as a companion
    output
- name: Kraken2
  role: Second taxonomic profiler run on cleaned non-host reads
- name: YongxinLiu/Liu2020ProteinCell
  role: Repository with the step-by-step reproducible analysis scripts implementing
    the pipeline
  repo: YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:4c31eb4a227fbe776a81d8492711d3541daf039251000681b72531a62447193c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - Bowtie2
  - HUMAnN2
  - KneadData
  - Kraken2
  - MetaPhlAn2
  - Trimmomatic
  - YongxinLiu/Liu2020ProteinCell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# kegg-orthology-functional-annotation

## Summary

Generates functional profiles of metagenomic samples with HUMAnN2 and regresses gene-family abundances into KEGG orthology (KO) tables, following the metagenomic pipeline recommended by Liu et al. 2020 for converting raw sequencing reads into taxonomic and functional feature tables.

## When to use

Use when raw metagenomic FASTQ reads must be converted into functional feature tables, specifically KEGG orthology (KO) abundance tables alongside taxonomic profiles, per the Liu2020ProteinCell reproducible pipeline.

## When NOT to use

- Input is already a taxonomic or functional feature table (the skill operates on raw metagenomic FASTQ reads)
- The goal is amplicon (16S) analysis rather than shotgun metagenomics — the review recommends separate pipelines for amplicon and metagenomic analyses
- The study targets metatranscriptome, metaproteome, or metabolome functional characterization, which the review notes are not widely accepted due to high cost and complex experimental/analysis methods

## Inputs

- raw metagenomic FASTQ files (paired-end sequencing reads)
- host reference genome/genome index for Bowtie2 host-read removal
- step-by-step reproducible analysis scripts from the GitHub repository YongxinLiu/Liu2020ProteinCell

## Outputs

- KEGG orthology (KO) gene-family abundance table per sample (functional profile table)
- HUMAnN2 gene-family functional profiles
- taxonomic abundance profile tables from MetaPhlAn2 and Kraken2 (companion output)
- cleaned non-host reads from KneadData for downstream profiling

## How to apply

First clean raw metagenomic FASTQ files with KneadData, which chains Trimmomatic adapter/quality trimming with Bowtie2 alignment against the host reference to remove host-contaminating reads. Run HUMAnN2 on the cleaned non-host reads to produce functional profiles, then regress gene families to KEGG orthology (KO) abundance tables; optionally run MetaPhlAn2 and Kraken2 on the same cleaned reads for taxonomic abundance profiles. Collect the taxonomic profile tables and the KO functional tables as the final pipeline outputs. The reproducible step-by-step scripts are obtained from the GitHub repository YongxinLiu/Liu2020ProteinCell accompanying the review's analysis guide.

## Related tools

- **KneadData** (Quality control and host-read removal: chains Trimmomatic trimming with Bowtie2 host alignment to output clean reads for downstream functional profiling)
- **Trimmomatic** (Adapter/quality trimming of raw FASTQ reads as part of the KneadData cleaning step)
- **Bowtie2** (Aligns reads against the host reference to remove host-contaminating reads during KneadData cleaning)
- **HUMAnN2** (Produces functional profiles from cleaned reads and regresses gene families to KEGG orthology (KO) abundance tables)
- **MetaPhlAn2** (Generates taxonomic abundance profiles from cleaned non-host reads as a companion output)
- **Kraken2** (Second taxonomic profiler run on cleaned non-host reads)
- **YongxinLiu/Liu2020ProteinCell** (Repository with the step-by-step reproducible analysis scripts implementing the pipeline) — YongxinLiu/Liu2020ProteinCell

## Evaluation signals

- The KEGG-orthology functional output table exists, is non-empty, and contains rows for all analyzed samples (the section text reports no quantitative acceptance criterion, so success is judged by table existence and sample coverage)
- The companion taxonomic profile tables (MetaPhlAn2/Kraken2) also exist and are non-empty for the same samples
- HUMAnN2 gene-family output was successfully regressed to KO abundances rather than left at gene-family level only
- Cleaned reads show host reads removed, confirming the KneadData (Trimmomatic + Bowtie2) step completed before profiling

## Limitations

- The section text reports no quantitative acceptance criterion, so pipeline success is validated only by checking that output tables exist, are non-empty, and contain rows for the analyzed samples
- The article's extracted entities do not specify tool versions or KO regression parameters, so exact thresholds must be taken from the YongxinLiu/Liu2020ProteinCell scripts
- Functional interpretation beyond gene families/KO (e.g., metatranscriptome, metaproteome, metabolome) is out of scope per the review

## Evidence

- [abstract] Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools.: "Then, we recommend specific pipelines for amplicon and metagenomic analyses, and describe commonly-used software and databases, to help researchers select the appropriate tools."
- [abstract] Finally, a step-by-step reproducible analysis guide is introduced.: "Finally, a step-by-step reproducible analysis guide is introduced."
- [readme] Pipeline & Reproducible analysis examples for amplicon and metagenome in microbiome: "Pipeline & Reproducible analysis examples for amplicon and metagenome in microbiome"
