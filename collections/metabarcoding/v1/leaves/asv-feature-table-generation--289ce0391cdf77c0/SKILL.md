---
name: asv-feature-table-generation
description: Use when raw 16S sequencing reads (downloaded from SRA) must be converted
  into a denoised ASV feature table and singleton reads must be retained for downstream
  alpha diversity metrics.
when_to_use_negative:
- When the goal is to remove singletons before feature-table construction (e.g. if
  downstream metrics do not require singletons), DADA2 denoising is the wrong substitute
  since it removes singletons and DEBLUR keeps them
- When input is already a denoised feature table rather than raw sequencing reads
- When comparing denoiser behavior on paired-end samples where one mate is missing
  — samples identified as paired-end that only had a single read should be removed
  first
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: sra-tools
  role: Downloads the raw 16S sequencing files for each dataset from the NCBI SRA
    using accession numbers from each dataset's folder
  repo: https://github.com/ncbi/sra-tools
- name: QIIME2 2022.2.0
  role: Analysis environment in which reads are imported, QC'd, denoised, and the
    feature table is produced following QIIME2 best practices
  repo: https://qiime2.org/
- name: DEBLUR
  role: Denoiser applied to QC'd reads to perform noise removal, dereplication, chimera
    removal, and ASV selection while preserving singletons
- name: anaconda
  role: Virtual environment manager hosting the QIIME2 2022.2.0 installation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:55738138c7e295321a91aa796afacbc5291610fccbeb1ff5afcd9f9c0d128b78
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ncbi/sra-tools
  tools:
  - DEBLUR
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# asv-feature-table-generation

## Summary

Generates an amplicon sequence variant (ASV) feature table from raw 16S FASTQ reads by running DEBLUR denoising inside QIIME2, which performs noise removal, dereplication, chimera removal, and ASV selection while preserving singletons.

## When to use

Use when raw 16S sequencing reads (downloaded from SRA) must be converted into a denoised ASV feature table and singleton reads must be retained for downstream alpha diversity metrics.

## When NOT to use

- When the goal is to remove singletons before feature-table construction (e.g. if downstream metrics do not require singletons), DADA2 denoising is the wrong substitute since it removes singletons and DEBLUR keeps them
- When input is already a denoised feature table rather than raw sequencing reads
- When comparing denoiser behavior on paired-end samples where one mate is missing — samples identified as paired-end that only had a single read should be removed first

## Inputs

- Raw 16S sequencing files (FASTQ) retrieved from NCBI SRA using sra-tools
- SRA accession numbers from each dataset's folder in the repository (SraRunTable.txt or similar)

## Outputs

- DEBLUR-denoised ASV feature table exported as feature-table.deblur.qza (non-rarefied, singletons retained)

## How to apply

Download the raw 16S sequencing files for each project from NCBI SRA using sra-tools, taking the accession numbers from each dataset's folder (e.g. ./datasets/xx_dataset_alias/SraRunTable.txt) in the repository. Import the reads into a QIIME2 2022.2.0 environment installed on an anaconda virtual environment, and perform per-dataset quality control (read quality assessment and adapter trimming) prior to denoising. Apply the DEBLUR denoiser to the QC'd reads to carry out noise removal, dereplication, chimera removal, and ASV selection; DEBLUR keeps singletons by default, unlike DADA2 which removes them, and singletons are required inputs for some alpha diversity metrics. Export the denoised feature table as feature-table.deblur.qza for use as the non-rarefied input to downstream alpha diversity calculations.

## Related tools

- **sra-tools** (Downloads the raw 16S sequencing files for each dataset from the NCBI SRA using accession numbers from each dataset's folder) — https://github.com/ncbi/sra-tools
- **QIIME2 2022.2.0** (Analysis environment in which reads are imported, QC'd, denoised, and the feature table is produced following QIIME2 best practices) — https://qiime2.org/
- **DEBLUR** (Denoiser applied to QC'd reads to perform noise removal, dereplication, chimera removal, and ASV selection while preserving singletons)
- **anaconda** (Virtual environment manager hosting the QIIME2 2022.2.0 installation)

## Examples

```
qiime deblur denoise-16S --i-demultiplexed-seqs demux-paired-end.qza --p-trim-length -1 --o-table feature-table.deblur.qza --o-representative-sequences rep-seqs.deblur.qza --o-stats deblur-stats.qza
```

## Evaluation signals

- The exported artifact is a valid QIIME2 .qza feature table (feature-table.deblur.qza) that can be loaded by QIIME2's diversity module
- Singleton sequences are present in the resulting feature table (DEBLUR retains them by default, unlike DADA2)
- The feature table was generated from reads that passed per-dataset QC (read quality assessment and adapter trimming) prior to denoising
- The workflow was executed in the documented environment (QIIME 2022.2.0 on an anaconda virtual environment), matching the commands under pipeline_scripts/workflow.txt
- Samples identified as paired-end that only had a single read were removed before denoising

## Limitations

- Raw sample data is not shared in the repository due to size; only accession numbers are available within each dataset's folder
- Samples identified as paired-end that only had a single read were removed, so such reads cannot be denoised
- Projects were excluded if they lacked associated metadata, which constrains which datasets can flow through the full pipeline
- DADA2 and DEBLUR differ in singleton treatment (DADA2 removes singletons, DEBLUR keeps them), so the denoise tool is a source of variability when reporting estimated microorganism richness and must be reported

## Evidence

- [intro] Each dataset was downloaded from the SRA using sra-tools https://github.com/ncbi/sra-tools.: "Each dataset was downloaded from the SRA using sra-tools https://github.com/ncbi/sra-tools."
- [discussion] DEBLUR keeps singletons while DADA2 removes them: "DADA2 removes singletons and DEBLUR keeps them."
- [results] All samples were processed with both DADA2 and DEBLUR: "all samples were processed using DADA2 and DEBLUR"
- [discussion] QC per dataset before denoising: "After performing QC for each dataset independently (read quality, adapter trimming)"
- [readme] README: QIIME2 processing on anaconda with accession numbers in dataset folders: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] README: workflow template and QIIME version: "A template of the commandas ran can be found under pipeline_scipts/workflow.txt. Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
