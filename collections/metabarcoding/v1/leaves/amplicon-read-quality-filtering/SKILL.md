---
name: amplicon-read-quality-filtering
description: Use when raw 16S rRNA amplicon FASTQ reads (e.g. Illumina MiSeq, ~300
  bp paired-end) need per-sample quality assessment, read counting, and denoising
  into ASVs prior to taxonomy assignment.
when_to_use_negative:
- Input is already a denoised ASV/feature table or genus-collapsed taxonomy table
  — no raw-read quality filtering is needed at that stage.
- Reads come from third-generation full-length 16S rRNA sequencing (e.g. Nanopore)
  rather than the ~300 bp Illumina MiSeq short-read data this pipeline was designed
  for.
- The goal is differential-variant analysis where the article found DADA2 'identified
  more real variants and output sequences than other methods' — consider DADA2 instead
  of Deblur for variant detection.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: FastQC
  role: Assesses per-base quality (Q30) and reports reads per sample, bases per sequence,
    sequence distribution length, and scores per sequence for the raw FASTQ files
- name: Deblur
  role: Denoises the filtered paired-end reads into bacterial ASVs and removes non-redundant
    reads
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Greengenes
  role: Reference database for genus-level taxonomy assignment after quality filtering
    and denoising
- name: Illumina MiSeq
  role: Sequencing platform producing the ~300 bp amplicon reads that the quality-filtering
    pipeline ingests
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:831395d9e24bb69bd05d1731df44cc8a99e3b0b16f0471058b7997fe36201d08
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - FastQC
  - Greengenes
  - Illumina MiSeq
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# amplicon-read-quality-filtering

## Summary

Quality-control and preprocessing workflow for 16S rRNA amplicon FASTQ reads from Illumina MiSeq sequencing: assess per-base quality with FastQC (Q30 criterion), filter raw reads, then denoise with Deblur and flag chimeras against the QIIME 2 non-chimeric 16S rRNA database before taxonomic assignment.

## When to use

Use when raw 16S rRNA amplicon FASTQ reads (e.g. Illumina MiSeq, ~300 bp paired-end) need per-sample quality assessment, read counting, and denoising into ASVs prior to taxonomy assignment.

## When NOT to use

- Input is already a denoised ASV/feature table or genus-collapsed taxonomy table — no raw-read quality filtering is needed at that stage.
- Reads come from third-generation full-length 16S rRNA sequencing (e.g. Nanopore) rather than the ~300 bp Illumina MiSeq short-read data this pipeline was designed for.
- The goal is differential-variant analysis where the article found DADA2 'identified more real variants and output sequences than other methods' — consider DADA2 instead of Deblur for variant detection.

## Inputs

- Raw paired-end FASTQ files from Illumina MiSeq 16S rRNA amplicon sequencing (33 sputum samples across V1–V2, V3–V4, V5–V7, V7–V9 regions)
- QIIME 2 non-chimeric 16S rRNA reference database for chimera flagging

## Outputs

- Per-sample read counts and quality metrics (Q30, bases per sequence, sequence length distribution, per-sequence scores) from FastQC
- Denoised, non-chimeric ASV table (~100,000 sequences/sample; 1,568,227 ASVs per hypervariable region)
- Genus-level collapsed taxonomy assignments via Greengenes (114 taxonomies), e.g. aggregated into preprocessing_summary.csv

## How to apply

Run FastQC (Version 0.11.9) on the raw FASTQ files to obtain Q30-quality sequences and tabulate reads per sample, bases per sequence, sequence distribution length, and per-sequence scores; the article applied this to FASTQ files from 33 sputum samples. Filter raw reads with the quality cut-off (the paper reports a cut-off score of up to 3011) before downstream processing. Denoise with the Deblur pipeline (intended for paired-end data), removing non-redundant reads, and identify chimeras using the QIIME 2 non-chimeric 16S rRNA database with a best-hit similarity rate below 99% as the flagging criterion. Assign taxonomy with the Greengenes database collapsed at the genus level and report per-sample depth (~100,000 sequences/sample), ASV counts (1,568,227 per hypervariable region), and taxonomy counts (114) for the V1–V2, V3–V4, V5–V7 and V7–V9 region combinations.

## Related tools

- **FastQC** (Assesses per-base quality (Q30) and reports reads per sample, bases per sequence, sequence distribution length, and scores per sequence for the raw FASTQ files)
- **Deblur** (Denoises the filtered paired-end reads into bacterial ASVs and removes non-redundant reads) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Greengenes** (Reference database for genus-level taxonomy assignment after quality filtering and denoising)
- **Illumina MiSeq** (Sequencing platform producing the ~300 bp amplicon reads that the quality-filtering pipeline ingests)

## Examples

```
fastqc -o qc_results raw_fastq/*.fastq.gz
```

## Evaluation signals

- Per-sample read depth after filtering averages ~100,000 sequences across all 33 sputum samples; flag any sample deviating substantially from this range.
- Total ASV count per hypervariable region reproduces the reported 1,568,227 ASVs; per-region counts that deviate from the reported figures should be flagged.
- Distinct taxonomy count after Greengenes genus-level collapsing equals 114.
- FastQC output confirms Q30-quality sequences and nonzero reads-per-sample tables for every sample before denoising.
- Chimera flagging uses the QIIME 2 non-chimeric 16S rRNA database with best-hit similarity < 99%; chimeric reads must be excluded from the final ASV table.

## Limitations

- The Illumina MiSeq 300 bp length limit constrains 16S rRNA profiling of longer hypervariable regions.
- The regions were trained with Greengenes despite the availability of more updated versions of SILVA, which can affect taxonomy counts.
- DADA2 identified more real variants and output sequences than other methods in the article's comparison, so Deblur-based counts may under-report variants.
- The repository README provided with the code is essentially empty, so exact CLI parameters beyond those stated in the methods must be recovered from the paper.

## Evidence

- [results] FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample, bases per sequence, sequence distribution length, and scores per sequence: "FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample, bases per sequence"
- [methods] FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011: "FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011"
- [methods] Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [methods] The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
