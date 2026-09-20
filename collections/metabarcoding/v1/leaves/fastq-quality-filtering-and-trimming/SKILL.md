---
name: fastq-quality-filtering-and-trimming
description: Use when raw paired-end 16S amplicon FASTQ files must be quality-screened
  (Q30) and length-uniformly prepared before a denoising step such as Deblur that
  expects short fragments of consistent length.
when_to_use_negative:
- Input is already a denoised ASV feature table (BIOM) or representative sequences
  — filtering operates on raw FASTQ reads only
- Reads come from long-read platforms (e.g. Nanopore full-length 16S rRNA sequencing)
  — this workflow targets short Illumina MiSeq fragments within the 300 bp platform
  limit
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: FastQC
  role: Assesses per-base quality, reads per sample, bases per sequence, sequence
    length distribution and scores per sequence; used to enforce the Q30 filtering
    cut-off
  repo: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Deblur
  role: Downstream denoising consumer of the filtered/trimmed reads; its preference
    for shorter, uniform-length paired-end fragments motivates the trimming parameters
  repo: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:478978269f9a05df522a1e8d870f48b9993ddec519b12a1ece881f200fd73aa8
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
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# fastq-quality-filtering-and-trimming

## Summary

Quality assessment and filtering of raw 16S amplicon FASTQ reads (2 x 300 bp MiSeq from V1–V2, V3–V4, V5–V7 and V7–V9 library pools) prior to Deblur ASV denoising, retaining Q30 sequences and verifying per-base quality, read counts and length distribution with FastQC.

## When to use

Use when raw paired-end 16S amplicon FASTQ files must be quality-screened (Q30) and length-uniformly prepared before a denoising step such as Deblur that expects short fragments of consistent length.

## When NOT to use

- Input is already a denoised ASV feature table (BIOM) or representative sequences — filtering operates on raw FASTQ reads only
- Reads come from long-read platforms (e.g. Nanopore full-length 16S rRNA sequencing) — this workflow targets short Illumina MiSeq fragments within the 300 bp platform limit

## Inputs

- Paired-end Illumina MiSeq FASTQ files (2 x 300 bp reads) from the 16S hypervariable region library pools (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- Quality-filtered (Q30) and length-trimmed FASTQ reads suitable for Deblur denoising
- FastQC per-sample quality reports (reads per sample, bases per sequence, sequence length distribution, per-sequence scores)

## How to apply

Run FastQC (Version 0.11.9 in this study) on the paired-end FASTQ files to observe reads per sample, bases per sequence, sequence length distribution and per-sequence quality scores. Filter the raw reads with a quality cut-off of Q30, as enforced via FastQC assessment, to retain high-quality sequences suitable for downstream denoising. Trim reads to a uniform length compatible with Deblur, which is recommended for shorter bp fragments and is intended for paired-end data. Confirm from the FastQC reports that per-base quality is acceptable and read counts are consistent before proceeding to denoising; poor-quality bases or highly variable length distributions indicate the filtering/trimming must be repeated before Deblur can infer ASVs reliably.

## Related tools

- **FastQC** (Assesses per-base quality, reads per sample, bases per sequence, sequence length distribution and scores per sequence; used to enforce the Q30 filtering cut-off) — https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Deblur** (Downstream denoising consumer of the filtered/trimmed reads; its preference for shorter, uniform-length paired-end fragments motivates the trimming parameters) — https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification

## Examples

```
fastqc *.fastq.gz --extract -o fastqc_reports  # verify Q30, reads/sample and length distribution before Deblur trimming
```

## Evaluation signals

- FastQC reports confirm retained sequences meet the Q30 quality cut-off
- Read counts per sample remain consistent and traceable after filtering
- Trimmed reads show a uniform sequence length distribution appropriate for Deblur's shorter-fragment error model
- Downstream Deblur denoising proceeds without failures attributable to residual low-quality or variable-length reads
- Chimera screening against the QIIME 2 non-chimeric 16S rRNA database (≥99% best-hit similarity) does not flag excess read loss, indicating over-filtering was avoided

## Limitations

- The 300 bp length limit of the Illumina MiSeq platform constrains 16S rRNA profiling, so filtering cannot recover information beyond what the short reads provide
- FastQC-based Q30 filtering does not itself remove chimeras; a separate QIIME 2 non-chimeric database screen with a <99% best-hit similarity threshold is required afterwards
- Quality scores alone do not guarantee correct taxonomic assignment downstream — Greengenes-trained classifiers and outdated reference versions can still bias results

## Evidence

- [results] FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample, bases per sequence, sequence distribution length, and scores per sequence: "FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample, bases per sequence, sequence distribution length, and scores per sequence"
- [methods] FastQC filtering cut-off: "FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011"
- [discussion] Deblur short-fragment rationale: "despite Deblur being recommended for shorter bp fragments"
- [methods] Denoising consumer context: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [methods] Chimera screen downstream check: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
