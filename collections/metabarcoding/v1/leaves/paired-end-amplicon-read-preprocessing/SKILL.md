---
name: paired-end-amplicon-read-preprocessing
description: Use when raw paired-end 16S amplicon FASTQ files (e.g. Illumina MiSeq
  V1–V2/V3–V4/V5–V7/V7–V9 libraries) must be quality-filtered, denoised into ASVs,
  and de-chimerized before genus-level taxonomic assignment.
when_to_use_negative:
- Input is already a denoised ASV feature table or BIOM file — re-running Deblur would
  be redundant
- Reads are full-length 16S rRNA sequences from third-generation/Nanopore platforms,
  since Deblur is recommended for shorter bp fragments and this workflow was tuned
  for 2 x 300 bp MiSeq data
- The goal is to maximize the number of recovered real variants — the article's discussion
  notes DADA2 identified more real variants and output sequences than other methods
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: FastQC
  role: Per-base quality assessment and Q30 filtering of raw paired-end reads before
    denoising
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Deblur
  role: Denoises paired-end 16S reads (recommended for shorter fragments) to infer
    ASVs and delete non-redundant reads
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: QIIME 2
  role: Provides the non-chimeric 16S rRNA database used to identify and remove chimeric
    reads at a >=99% best-hit similarity threshold
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:7850f424fc9a95aa2b5368bbdfb23c603139de19eede96b8329642f7ec1771bd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - Deblur
  - FastQC
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# paired-end-amplicon-read-preprocessing

## Summary

A quality-control and denoising workflow for paired-end 16S rRNA amplicon FASTQ reads (2 x 300 bp MiSeq) that uses FastQC for Q30-based quality assessment, the Deblur pipeline to infer amplicon sequence variants (ASVs) from short fragments, and QIIME 2's non-chimeric 16S database for chimera removal, yielding a denoised ASV feature table for genus-level taxonomy.

## When to use

Use when raw paired-end 16S amplicon FASTQ files (e.g. Illumina MiSeq V1–V2/V3–V4/V5–V7/V7–V9 libraries) must be quality-filtered, denoised into ASVs, and de-chimerized before genus-level taxonomic assignment.

## When NOT to use

- Input is already a denoised ASV feature table or BIOM file — re-running Deblur would be redundant
- Reads are full-length 16S rRNA sequences from third-generation/Nanopore platforms, since Deblur is recommended for shorter bp fragments and this workflow was tuned for 2 x 300 bp MiSeq data
- The goal is to maximize the number of recovered real variants — the article's discussion notes DADA2 identified more real variants and output sequences than other methods

## Inputs

- Paired-end FASTQ files (2 x 300 bp Illumina MiSeq 16S amplicon reads)
- Non-chimeric 16S rRNA reference database from QIIME 2

## Outputs

- Denoised ASV feature table in BIOM format
- Representative ASV sequences for downstream taxonomic assignment
- FastQC quality reports (reads per sample, per-base scores, sequence length distribution)

## How to apply

First run FastQC (Version 0.11.9 in the source study) on the raw paired-end reads to verify per-base quality, reads per sample, and sequence length distribution, then quality-filter reads to Q30. Denoise the paired-end reads with the Deblur pipeline, which is recommended for shorter bp fragments and deletes non-redundant reads while inferring ASVs via its static error model rather than OTU binning. Screen the denoised reads against the non-chimeric 16S rRNA database from QIIME 2, removing reads whose best hit similarity rate is less than 99% (i.e. candidate chimeras). Finally, export the denoised ASV feature table (BIOM format) with representative sequences as input for taxonomic assignment collapsed at the genus level using Greengenes.

## Related tools

- **FastQC** (Per-base quality assessment and Q30 filtering of raw paired-end reads before denoising) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Deblur** (Denoises paired-end 16S reads (recommended for shorter fragments) to infer ASVs and delete non-redundant reads) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **QIIME 2** (Provides the non-chimeric 16S rRNA database used to identify and remove chimeric reads at a >=99% best-hit similarity threshold)

## Evaluation signals

- FastQC reports show reads filtered to Q30 and per-base quality scores above the stated cutoff before denoising
- Denoised read set contains no non-redundant reads, consistent with Deblur's deletion of redundant sequences
- Chimera screening retains only reads with best-hit similarity rate >= 99% against the QIIME 2 non-chimeric 16S rRNA database
- Output ASV feature table is exported in BIOM format with representative sequences ready for Greengenes genus-level collapsing
- Mock community (ZymoBIOMICS) control processed through the same workflow yields expected genus-level taxa

## Limitations

- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling, constraining region resolution
- Deblur is recommended for shorter bp fragments, so very long amplicons may be better served by other methods
- The study's discussion notes DADA2 identified more real variants and output sequences than other methods, so Deblur may under-detect true variants

## Evidence

- [methods] Deblur denoising of paired-end reads: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [methods] FastQC quality filtering: "FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011"
- [methods] Chimera removal threshold: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [abstract] Deblur ASV identification: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
- [discussion] Deblur suitability for short fragments: "despite Deblur being recommended for shorter bp fragments"
