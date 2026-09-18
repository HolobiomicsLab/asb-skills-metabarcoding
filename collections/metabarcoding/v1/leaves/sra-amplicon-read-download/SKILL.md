---
name: sra-amplicon-read-download
description: Use when raw 16S reads must be fetched from SRA for a published microbiome
  study that only shares accession numbers (e.g. the 13 stool projects behind 10.5281/zenodo.8170289)
  rather than FASTQ files.
when_to_use_negative:
- The raw FASTQ files are already available locally or in the analysis repository
  (the authors explicitly do not share raw sample data due to size, so re-download
  is only needed when starting from scratch).
- The data source is EMBL or another repository rather than NCBI SRA; this skill only
  covers SRA retrieval via sra-tools.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: sra-tools
  role: Downloads each dataset's raw 16S sequencing files from NCBI SRA using the
    accession numbers listed in the study's Zenodo-deposited SraRunTable.txt files
  repo: https://github.com/ncbi/sra-tools
merged_aliases:
- sra-amplicon-read-retrieval
merged_alias_records:
- alias: sra-amplicon-read-retrieval
  slug: sra-amplicon-read-retrieval
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:ec467e7e9db4b436800b80f50d98322ea969f900ae66614326b47a34b56effc6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncbi/sra-tools
  tools:
  - DEBLUR
  - QIIME 2
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# sra-amplicon-read-download

## Summary

Retrieve raw 16S amplicon sequencing files from NCBI SRA using sra-tools, with accession numbers recovered from each dataset's folder (SraRunTable.txt) in the authors' Zenodo deposit, as the first step of the alpha-diversity reanalysis pipeline.

## When to use

Use when raw 16S reads must be fetched from SRA for a published microbiome study that only shares accession numbers (e.g. the 13 stool projects behind 10.5281/zenodo.8170289) rather than FASTQ files.

## When NOT to use

- The raw FASTQ files are already available locally or in the analysis repository (the authors explicitly do not share raw sample data due to size, so re-download is only needed when starting from scratch).
- The data source is EMBL or another repository rather than NCBI SRA; this skill only covers SRA retrieval via sra-tools.

## Inputs

- SRA accession numbers per dataset (SraRunTable.txt files from ./datasets/xx_dataset_alias/ of Zenodo 10.5281/zenodo.8170289)
- NCBI sra-tools installation (SRA Toolkit)

## Outputs

- Raw 16S paired-end sequencing files (FASTQ) for each of the 13 publicly available human microbiome projects, downloaded per project from SRA

## How to apply

Locate each study's accession list in the deposit (Zenodo DOI 10.5281/zenodo.8170289), where the authors provide SraRunTable.txt files per dataset under ./datasets/xx_dataset_alias/ because raw reads are too large to host. Download the raw sequencing files from NCBI SRA with the NCBI sra-tools toolkit installed in the analysis environment. Keep downloads organized per project so that downstream QIIME2 2022.2.0 import, per-dataset QC (read quality assessment, adapter trimming), and DEBLUR denoising can be run independently for each dataset. Raw data must not be redistributed from the analysis repository; only the accession tables should be shared.

## Related tools

- **sra-tools** (Downloads each dataset's raw 16S sequencing files from NCBI SRA using the accession numbers listed in the study's Zenodo-deposited SraRunTable.txt files) — https://github.com/ncbi/sra-tools

## Evaluation signals

- Every accession listed in each dataset's SraRunTable.txt yields a downloaded sequencing file; no project from the 13 stool studies is missing.
- Downloaded reads are organized per dataset (one folder per xx_dataset_alias) matching the repository's datasets/ layout, ready for per-dataset QIIME2 import.
- The workflow covers all 4,596 stool samples from the 13 publicly available human microbiome projects before QC filtering.
- Raw reads are not committed back to the public analysis repository (only accession numbers are redistributed).

## Limitations

- Raw sample data is not shared through the authors' repository due to size — only accession numbers, so availability depends on the SRA entries remaining accessible.
- Projects lacking associated metadata are excluded downstream, so some downloaded reads may be discarded at the filtering stage.
- Paired-end samples that only had a single read are removed in later QC, so raw downloads will include samples that do not survive processing.

## Evidence

- [intro] Each dataset was downloaded from the SRA using sra-tools https://github.com/ncbi/sra-tools.: "Each dataset was downloaded from the SRA using sra-tools https://github.com/ncbi/sra-tools."
- [readme] Raw sample data is not shared through this repository due to their size, however the accession numbers are available within each dataset's folder: "Raw sample data is not shared through this repository due to their size, however the accession numbers are available within each dataset's folder"
- [readme] the accession numbers are available within each dataset's folder (./datasets/xx_dataset_alias/SraRunTable.txt or similar): "the accession numbers are available within each dataset's folder (./datasets/xx_dataset_alias/SraRunTable.txt or similar)"
