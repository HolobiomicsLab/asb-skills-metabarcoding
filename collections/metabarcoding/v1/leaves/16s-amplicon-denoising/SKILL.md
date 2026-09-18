---
name: 16s-amplicon-denoising
description: Use when raw 16S FASTQ reads from SRA must be converted into a per-sample
  ASV feature table that preserves singletons, e.g. to support singleton-dependent
  alpha diversity metrics such as Faith's phylogenetic diversity in QIIME2.
when_to_use_negative:
- Input is already a denoised feature table (e.g. an existing .qza) rather than raw
  sequencing reads from SRA
- The downstream analysis uses DADA2-processed data or metrics unaffected by singleton
  removal — applying DEBLUR-based denoising then changes richness estimates relative
  to DADA2
- Reads are shotgun metagenomic or non-16S amplicon data where DEBLUR's 16S error
  model does not apply
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: DEBLUR
  role: Denoiser performing noise removal, dereplication, chimera removal, and ASV
    selection while keeping singletons
- name: QIIME2 2022.2.0
  role: Framework in which DEBLUR denoising and QC are executed following QIIME2 best
    practices; commands templated in pipeline_scripts/workflow.txt
  repo: https://qiime2.org/
- name: sra-tools
  role: Downloads raw 16S sequencing files from NCBI SRA using accession numbers in
    each dataset's folder
  repo: https://github.com/ncbi/sra-tools
- name: anaconda
  role: Virtual environment hosting the QIIME2 2022.2.0 installation on Ubuntu
- name: DADA2
  role: Alternative denoiser run for comparison; removes singletons, so its feature
    table is not used for the downstream alpha diversity analyses
merged_aliases:
- deblur-denoising-of-16s-amplicons
merged_alias_records:
- alias: deblur-denoising-of-16s-amplicons
  slug: deblur-denoising-of-16s-amplicons
  cosine_score: 0.8138
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:16edcc6ff08ef34b4c9a3635f3b3e9ace8e8320ea4fa8721bbdbc201ec7116fd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - QIIME 2
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
yes_rate: 0.36363636363636365
n_claims: 11
---

# 16S-amplicon-denoising

## Summary

Denoises 16S rRNA amplicon reads with DEBLUR inside QIIME2 (2022.2.0), performing noise removal, dereplication, chimera removal, and ASV selection while retaining singletons — the choice of denoiser directly determines which alpha diversity metrics can be computed downstream.

## When to use

Use when raw 16S FASTQ reads from SRA must be converted into a per-sample ASV feature table that preserves singletons, e.g. to support singleton-dependent alpha diversity metrics such as Faith's phylogenetic diversity in QIIME2.

## When NOT to use

- Input is already a denoised feature table (e.g. an existing .qza) rather than raw sequencing reads from SRA
- The downstream analysis uses DADA2-processed data or metrics unaffected by singleton removal — applying DEBLUR-based denoising then changes richness estimates relative to DADA2
- Reads are shotgun metagenomic or non-16S amplicon data where DEBLUR's 16S error model does not apply

## Inputs

- Raw 16S sequencing files (FASTQ) retrieved from NCBI SRA using sra-tools, with accession numbers from each dataset's SraRunTable.txt
- Per-dataset QC'd reads (read quality assessed, adapters trimmed)
- Paired-end sample list with single-read samples removed

## Outputs

- feature-table.deblur.qza — DEBLUR-denoised per-sample ASV feature table including singletons
- Non-rarefied feature table serving as input to QIIME2 diversity-module alpha diversity calculations

## How to apply

Download each dataset's raw sequencing files from NCBI SRA with sra-tools, taking accession numbers from each dataset's folder (SraRunTable.txt) rather than expecting shared raw data. Import the reads into QIIME2 2022.2.0 running in an anaconda virtual environment and perform per-dataset QC (read quality assessment and adapter trimming), removing paired-end samples that only had a single read. Apply DEBLUR following QIIME2 best practices so that denoising, dereplication, chimera removal, and ASV selection occur in one stage; keep singletons, since DEBLUR retains them by default whereas DADA2 removes them. Export the denoised table as feature-table.deblur.qza and use it as the non-rarefied input to downstream alpha diversity calculations — process all samples with both DADA2 and DEBLUR only if you intend to compare denoiser effects on richness estimates.

## Related tools

- **DEBLUR** (Denoiser performing noise removal, dereplication, chimera removal, and ASV selection while keeping singletons)
- **QIIME2 2022.2.0** (Framework in which DEBLUR denoising and QC are executed following QIIME2 best practices; commands templated in pipeline_scripts/workflow.txt) — https://qiime2.org/
- **sra-tools** (Downloads raw 16S sequencing files from NCBI SRA using accession numbers in each dataset's folder) — https://github.com/ncbi/sra-tools
- **anaconda** (Virtual environment hosting the QIIME2 2022.2.0 installation on Ubuntu)
- **DADA2** (Alternative denoiser run for comparison; removes singletons, so its feature table is not used for the downstream alpha diversity analyses)

## Examples

```
qiime deblur denoise-16S --i-demultiplexed-seqs demux.qza --p-trim-length -1 --p-sample-stats --o-table feature-table.deblur.qza --o-representative-sequences rep-seqs.deblur.qza --o-stats deblur.stats.qza
```

## Evaluation signals

- Exported table is named feature-table.deblur.qza and is used as the non-rarefied input for all downstream alpha diversity calculations
- Singletons are present in the denoised feature table (DEBLUR keeps them), verified against a DADA2-denoised comparison table where singletons are absent
- Aggregated per-sample table (aggregated.tsv) covers the 4,596 stool samples from the 13 publicly available human microbiome projects with all 19 alpha diversity metrics as columns
- Paired-end samples that only had a single read were removed during QC before denoising
- Re-run on rarefied datasets reproduces the same qualitative results as the non-rarefied run, as done for validation

## Limitations

- DADA2 and DEBLUR differ in singleton treatment (DADA2 removes singletons, DEBLUR keeps them), producing different richness estimates; the denoise tool must be reported as a source of variability when reporting estimated microorganism richness
- No version numbers were reported for DADA2 or DEBLUR, complicating exact reproducibility
- Raw sample data are not shared in the repository due to size; only accession numbers (SraRunTable.txt per dataset folder) are provided
- DEBLUR must be run after per-dataset QC (read quality assessment, adapter trimming); skipping QC or applying it globally rather than per dataset is not validated

## Evidence

- [discussion] DADA2 removes singletons and DEBLUR keeps them.: "DADA223 and DEBLUR24 not only have different algorithmic denoising techniques but also have relevant differences in their singletons treatment approach."
- [readme] All sample processing was done using QIIME2 on an anaconda virtual environment; datasets downloaded from SRA with sra-tools.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] Accession numbers are available within each dataset's folder (SraRunTable.txt) since raw data are too large to share.: "Raw sample data is not shared through this repository due to their size, however the accession numbers are available within each dataset's folder"
- [readme] Workflows were run on QIIME 2022.2.0, commands templated under pipeline_scripts/workflow.txt.: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [discussion] Paired-end samples with only a single read were removed during QC.: "Samples identified as paired-end that only had a single read were removed."
- [results] All samples were processed using both DADA2 and DEBLUR.: "all samples were processed using DADA2 and DEBLUR"
