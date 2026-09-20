---
name: asv-denoising-count-summary
description: Use when raw FASTQ reads from 16S rRNA amplicon samples need to be quality-filtered,
  denoised to ASVs, and summarized into per-region sequence/ASV/taxonomy counts to
  validate reported preprocessing figures.
when_to_use_negative:
- Input is already a denoised ASV feature table or post-QC count matrix — re-running
  Deblur denoising would double-process the data
- Long-read or full-length 16S rRNA data (e.g., Nanopore) — Deblur is intended for
  paired-end data and the 300 bp Illumina MiSeq limit is a stated constraint of this
  workflow
- Goal is downstream diversity or ROC/sensitivity analysis rather than reproducing
  preprocessing counts
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: FastQC
  role: Assesses quality and filters raw reads (Version 0.11.9, Q30 criterion) and
    tabulates the number of reads per sample
- name: Deblur
  role: Denoises paired-end sequences, deletes non-redundant reads, and identifies
    bacterial ASVs at the genus level
- name: QIIME 2
  role: Provides the non-chimeric 16S rRNA database for chimera identification (<99%
    best-hit similarity) and runs the Greengenes genus-level taxonomic collapsing
- name: Greengenes
  role: Reference database for taxonomic assignments collapsed at the genus level
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:54dc70efed2ee93387f1074769832a72cd3946bd8ca60f26557be699f988a55d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - FastQC
  - Greengenes
  - Illumina MiSeq
  - QIIME 2
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# asv-denoising-count-summary

## Summary

Reproduces the reported preprocessing and ASV/taxonomy counts for the 33 sputum samples in a 16S rRNA hypervariable-region comparison by quality-filtering raw reads with FastQC (Q30), denoising with Deblur, flagging chimeras against the QIIME 2 non-chimeric 16S rRNA database, and aggregating counts per region (V1–V2, V3–V4, V5–V7, V7–V9).

## When to use

Use when raw FASTQ reads from 16S rRNA amplicon samples need to be quality-filtered, denoised to ASVs, and summarized into per-region sequence/ASV/taxonomy counts to validate reported preprocessing figures.

## When NOT to use

- Input is already a denoised ASV feature table or post-QC count matrix — re-running Deblur denoising would double-process the data
- Long-read or full-length 16S rRNA data (e.g., Nanopore) — Deblur is intended for paired-end data and the 300 bp Illumina MiSeq limit is a stated constraint of this workflow
- Goal is downstream diversity or ROC/sensitivity analysis rather than reproducing preprocessing counts

## Inputs

- Raw fastq read files from the 33 sputum samples (deposited at github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification)
- Hypervariable region combination labels (V1–V2, V3–V4, V5–V7, V7–V9)
- Greengenes reference database for genus-level taxonomic assignment

## Outputs

- Per-sample read counts tabulated by FastQC under the Q30 criterion
- ASV counts per hypervariable region after Deblur denoising and chimera flagging
- Distinct taxonomy counts per region (114 taxonomies reported)
- preprocessing_summary.csv aggregating sequences per sample, ASVs per region, and taxonomy counts

## How to apply

Obtain the raw fastq read files for the 33 sputum samples from the study's GitHub repository, then assess quality and tabulate reads per sample with FastQC (Version 0.11.9) using the Q30 quality criterion. Denoise the paired-end sequences with the Deblur pipeline, deleting non-redundant reads, and flag chimeras against the QIIME 2 non-chimeric 16S rRNA database at a best-hit similarity rate of less than 99% to obtain ASV counts. Assign taxonomy with the Greengenes database collapsed at the genus level (QIIME 2) and count distinct taxonomies per region combination (V1–V2, V3–V4, V5–V7, V7–V9). Finally, aggregate the per-sample sequence counts (~100,000 sequences/sample), the total ASV count per region (1,568,227), and the taxonomy count (114) into a machine-readable table saved as preprocessing_summary.csv.

## Related tools

- **FastQC** (Assesses quality and filters raw reads (Version 0.11.9, Q30 criterion) and tabulates the number of reads per sample)
- **Deblur** (Denoises paired-end sequences, deletes non-redundant reads, and identifies bacterial ASVs at the genus level)
- **QIIME 2** (Provides the non-chimeric 16S rRNA database for chimera identification (<99% best-hit similarity) and runs the Greengenes genus-level taxonomic collapsing)
- **Greengenes** (Reference database for taxonomic assignments collapsed at the genus level)

## Examples

```
fastqc raw/*.fastq.gz && qiime deblur denoise-16S --i-demultiplexed-seqs seqs.qza --o-table table.qza --o-stats stats.qza
```

## Evaluation signals

- Average sequences per sample over the 33 sputum samples reproduces the reported ~100,000 sequences/sample
- Total ASV count per hypervariable region matches the reported 1,568,227; flag any region whose counts deviate
- Distinct taxonomy count per region combination equals the reported 114 taxonomies
- FastQC tabulated read counts are reproducible under the Q30 quality criterion with FastQC Version 0.11.9
- Chimera-flagged reads are consistent with the QIIME 2 non-chimeric 16S rRNA database at a best-hit similarity below 99%

## Limitations

- Deblur is recommended for shorter bp fragments, and the 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling
- Regions were trained with Greengenes despite the availability of more updated versions of SILVA, which may affect taxonomy counts
- The mock microbial community contained only 10 species, affecting sensitivity and specificity of taxonomic classification; no comparison to full-length 16S rRNA sequencing was performed

## Evidence

- [methods] FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011: "FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011"
- [methods] Deblur denoising of paired-end data: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [methods] Chimera identification thresholds: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [methods] Taxonomic assignment database: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [abstract] ASV identification with Deblur: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
- [results] FastQC Q30 read counts per sample: "FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample"
- [discussion] Illumina MiSeq length limitation: "the 300 bp length limit of the Illumna MiSeq platform limits 16S rRNA profiling"
- [readme] Repository title: "Hypervariable regions of 16S rRNA with the highest resolving power for taxonomic identification"
