---
name: chimera-removal-and-dereplication
description: Use when raw 16S reads (FASTQ from SRA) must be converted into an ASV
  feature table for alpha diversity metrics that require singletons, so DADA2's singleton
  removal is unacceptable.
when_to_use_negative:
- When the alpha metrics of interest do not need singletons and singleton removal
  is acceptable — DADA2 could be used instead, though the paper warns this changes
  estimated microorganism richness
- When input is already a denoised feature table (e.g. feature-table.deblur.qza) —
  re-denoising is unnecessary
- 'When raw reads are unavailable: the repository only shares accession numbers (in
  each dataset''s folder), so denoising cannot be reproduced without re-downloading
  from SRA'
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0196
tools:
- name: DEBLUR
  role: Denoiser applied to QC'd reads to perform noise removal, dereplication, chimera
    removal, and ASV selection while keeping singletons
- name: DADA2
  role: Alternative denoiser run on the same samples for comparison; rejected for
    downstream alpha diversity because it removes singletons
- name: QIIME2 2022.2.0
  role: Framework in which datasets were processed following QIIME2 best practices
    and whose diversity module consumes the DEBLUR feature table
  repo: https://qiime2.org/
- name: sra-tools
  role: Downloads the raw 16S sequencing files for each project from NCBI SRA prior
    to denoising
  repo: https://github.com/ncbi/sra-tools
- name: anaconda
  role: Provides the virtual environment hosting the QIIME2 2022.2.0 installation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:93a001be1d8bea2889160b853d97ca3bd0e91e4d7f2324243bad072065d4ec64
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# chimera-removal-and-dereplication

## Summary

Denoises 16S amplicon reads with DEBLUR inside QIIME2 2022.2.0, which jointly performs noise removal, dereplication, chimera removal, and ASV selection while retaining singletons. This skill exists because the paper's singleton-requiring alpha diversity metrics could only be computed on the DEBLUR feature table, not on DADA2 output, which discards singletons.

## When to use

Use when raw 16S reads (FASTQ from SRA) must be converted into an ASV feature table for alpha diversity metrics that require singletons, so DADA2's singleton removal is unacceptable.

## When NOT to use

- When the alpha metrics of interest do not need singletons and singleton removal is acceptable — DADA2 could be used instead, though the paper warns this changes estimated microorganism richness
- When input is already a denoised feature table (e.g. feature-table.deblur.qza) — re-denoising is unnecessary
- When raw reads are unavailable: the repository only shares accession numbers (in each dataset's folder), so denoising cannot be reproduced without re-downloading from SRA

## Inputs

- Raw 16S sequencing files (FASTQ) downloaded from NCBI SRA via sra-tools, with accession numbers from each dataset's SraRunTable.txt
- QC'd, demultiplexed QIIME2 sequence artifact (demux.qza) after read quality assessment and adapter trimming
- QIIME2 2022.2.0 environment installed on an anaconda virtual environment

## Outputs

- feature-table.deblur.qza — denoised, dereplicated, chimera-filtered ASV feature table retaining singletons
- Non-rarefied DEBLUR feature table used as input to QIIME2 diversity module alpha metrics

## How to apply

First import the SRA-derived reads into QIIME2 2022.2.0 (running in an anaconda virtual environment) and perform per-dataset quality control, including read quality assessment and adapter trimming. Then run the DEBLUR denoiser on the QC'd demultiplexed reads; DEBLUR carries out noise removal, dereplication, chimera removal, and ASV selection in one step and keeps singletons by default, unlike DADA2. Export the resulting denoised table as feature-table.deblur.qza and use it as the non-rarefied input for downstream alpha diversity calculations with QIIME2's diversity module. Do not switch denoisers downstream: the article shows DADA2 and DEBLUR produce different richness estimates because of their singleton treatment, so the denoise tool must be reported as a source of variability. If a comparison is desired, process the same samples with both DADA2 and DEBLUR but restrict all following analyses to the DEBLUR feature table.

## Related tools

- **DEBLUR** (Denoiser applied to QC'd reads to perform noise removal, dereplication, chimera removal, and ASV selection while keeping singletons)
- **DADA2** (Alternative denoiser run on the same samples for comparison; rejected for downstream alpha diversity because it removes singletons)
- **QIIME2 2022.2.0** (Framework in which datasets were processed following QIIME2 best practices and whose diversity module consumes the DEBLUR feature table) — https://qiime2.org/
- **sra-tools** (Downloads the raw 16S sequencing files for each project from NCBI SRA prior to denoising) — https://github.com/ncbi/sra-tools
- **anaconda** (Provides the virtual environment hosting the QIIME2 2022.2.0 installation)

## Examples

```
qiime deblur denoise-16S --i-demultiplexed-seqs demux.qza --p-trim-length -1 --o-table feature-table.deblur.qza --o-representative-sequences rep-seqs.deblur.qza
```

## Evaluation signals

- The DEBLUR step yields a feature-table.deblur.qza artifact in each dataset's directory that can be exported and fed to QIIME2's diversity module without rarefaction
- Singletons are present in the DEBLUR feature table (nonzero singleton counts), in contrast to a DADA2-processed table where they are removed
- All sample-processing commands match the QIIME2 best-practices template stored in pipeline_scripts/workflow.txt of the mauroibz/alpha-diversities repository
- The denoiser choice (DEBLUR, not DADA2) is explicitly reported downstream, since the article identifies the denoise tool as a source of variability for estimated richness
- Per-dataset QC (read quality assessment, adapter trimming) was completed before denoising, and paired-end samples that only had a single read were removed beforehand

## Limitations

- No version numbers were reported for DADA2 or DEBLUR (only QIIME2 2022.2.0 is stated), making exact reproduction of denoising behavior version-sensitive
- Denoiser choice is a source of variability: DADA2 and DEBLUR 'not only have different algorithmic denoising techniques but also have relevant differences in their singletons treatment approach'
- Raw sample data are not shared in the repository due to size, so the denoising stage must be re-run from SRA accession numbers in each dataset's folder

## Evidence

- [results] All samples processed using DADA2 and DEBLUR; downstream analyses performed with DEBLUR data: "all samples were processed using DADA2 and DEBLUR"
- [discussion] DADA2 removes singletons and DEBLUR keeps them: "DADA2 removes singletons and DEBLUR keeps them."
- [discussion] DADA2 and DEBLUR differ in algorithmic denoising and singleton treatment: "DADA223 and DEBLUR24 not only have different algorithmic denoising techniques but also have relevant differences in their singletons treatment approach."
- [readme] All sample processing was done in QIIME2 on anaconda; datasets downloaded with sra-tools: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment. 
Each dataset was downloaded from the SRA using sra-tools"
- [readme] Datasets processed following QIIME2 best practices with commands in /pipeline_scripts: "Each dataset was processed following QIIME2's best practices. All the commands used for sample processing can be found the /pipeline_scripts directory."
- [discussion] QC performed per dataset before denoising: "After performing QC for each dataset independently (read quality, adapter trimming)"
