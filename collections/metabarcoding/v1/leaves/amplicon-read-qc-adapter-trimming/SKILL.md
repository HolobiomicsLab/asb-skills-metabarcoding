---
name: amplicon-read-qc-adapter-trimming
description: Use when raw amplicon FASTQ reads have just been retrieved from SRA and
  must be quality-checked and adapter-trimmed per dataset before denoising with DEBLUR
  in QIIME2.
when_to_use_negative:
- Input is already a denoised QIIME2 feature table (post-DEBLUR/DADA2) rather than
  raw SRA reads
- Datasets will be denoised with DADA2 instead of DEBLUR — QC decisions such as single-read
  paired-end removal and singleton handling assumptions differ
- Study design requires pooled cross-study processing rather than per-dataset independent
  QC
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: sra-tools
  role: Downloads each dataset's raw sequencing reads from the SRA prior to QC
  repo: https://github.com/ncbi/sra-tools
- name: QIIME2 2022.2.0
  role: Runs per-dataset QC (read quality assessment, adapter trimming) following
    QIIME2 best practices in an anaconda environment
  repo: https://qiime2.org/
- name: DEBLUR
  role: Denoising step that consumes QC-passed trimmed reads to produce per-dataset
    feature tables
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:46e0cfff4d4a5479508608d46c79bc002947858259e7bcd3e9e3b96715c0f474
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncbi/sra-tools
  tools:
  - DEBLUR
  - QIIME 2
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# amplicon-read-qc-adapter-trimming

## Summary

Per-dataset quality control of amplicon sequencing reads (read-quality assessment plus adapter trimming) as a prerequisite step before DEBLUR denoising in the QIIME2 2022.2.0 alpha-diversity workflow over 4,596 stool samples from 13 human microbiome projects.

## When to use

Use when raw amplicon FASTQ reads have just been retrieved from SRA and must be quality-checked and adapter-trimmed per dataset before denoising with DEBLUR in QIIME2.

## When NOT to use

- Input is already a denoised QIIME2 feature table (post-DEBLUR/DADA2) rather than raw SRA reads
- Datasets will be denoised with DADA2 instead of DEBLUR — QC decisions such as single-read paired-end removal and singleton handling assumptions differ
- Study design requires pooled cross-study processing rather than per-dataset independent QC

## Inputs

- Raw paired-end amplicon FASTQ reads downloaded from SRA via sra-tools
- Per-dataset accession tables (SraRunTable.txt or similar) from each dataset's folder in the 10.5281/zenodo.8170289 deposit

## Outputs

- QC-passed, adapter-trimmed reads per dataset ready for DEBLUR denoising
- Per-dataset QIIME2 artifacts (imported reads, demux/QC objects) under each dataset's directory

## How to apply

Download each dataset's reads from SRA with sra-tools, then process each dataset independently (not pooled) following QIIME2's best practices: assess read quality, trim adapters, and remove paired-end samples that only had a single read. Commands should follow the template in pipeline_scripts/workflow.txt of the alpha-diversities repository, run on QIIME2 2022.2.0 in an anaconda virtual environment. The rationale is that per-dataset QC avoids cross-study batch effects and produces clean per-dataset feature tables that can subsequently be denoised with DEBLUR (which, unlike DADA2, keeps singletons) before computing the 19 alpha diversity metrics on non-rarefied data.

## Related tools

- **sra-tools** (Downloads each dataset's raw sequencing reads from the SRA prior to QC) — https://github.com/ncbi/sra-tools
- **QIIME2 2022.2.0** (Runs per-dataset QC (read quality assessment, adapter trimming) following QIIME2 best practices in an anaconda environment) — https://qiime2.org/
- **DEBLUR** (Denoising step that consumes QC-passed trimmed reads to produce per-dataset feature tables)

## Evaluation signals

- Paired-end samples identified as having only a single read are removed before denoising
- QC is performed independently per dataset (read quality, adapter trimming), not pooled across the 13 stool projects
- Commands match the template in pipeline_scripts/workflow.txt and run on QIIME2 2022.2.0
- QC output successfully denoises with DEBLUR, yielding per-dataset feature tables covering all 4,596 stool samples
- Downstream alpha diversity metrics computed on the resulting non-rarefied feature tables reproduce the aggregated alpha_aggregated.tsv structure

## Limitations

- Raw sample data is not shared in the Zenodo repository due to size; only accession numbers are provided, so QC depends on re-downloading from SRA
- Denoise-tool choice is a source of variability for estimated richness: DADA2 removes singletons while DEBLUR keeps them, so QC/denoising decisions affect diversity estimates
- Workflows were run on QIIME2 2022.2.0; compatibility with newer versions is expected but not guaranteed

## Evidence

- [discussion] Per-dataset QC rationale: "After performing QC for each dataset independently (read quality, adapter trimming)"
- [discussion] Single-read paired-end filter: "Samples identified as paired-end that only had a single read were removed."
- [readme] QIIME2 best-practices processing: "Each dataset was processed following QIIME2's best practices."
- [readme] Command templates location and version: "All the commands used for sample processing can be found the /pipeline_scripts directory."
- [readme] QIIME2 version compatibility note: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [intro] SRA retrieval step: "Each dataset was downloaded from the SRA using sra-tools https://github.com/ncbi/sra-tools."
