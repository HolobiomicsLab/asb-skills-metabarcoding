---
name: microbiome-sequence-count-tabulation
description: Use when raw 16S rRNA amplicon FASTQ files (e.g., 33 sputum samples sequenced
  on Illumina MiSeq) must be quality-filtered, denoised, and reduced to per-sample
  sequence counts, per-region ASV totals, and genus-level taxonomy counts.
when_to_use_negative:
- Input is already a denoised feature/ASV table or a collapsed genus-level abundance
  table — this skill regenerates counts from raw FASTQ, so it would duplicate completed
  preprocessing.
- The data are full-length 16S rRNA reads from third-generation platforms (e.g., Nanopore)
  — the pipeline is tuned to Illumina MiSeq's 300 bp length limit and paired-end Deblur.
- The goal is functional profiling or strain-level resolution — Greengenes is collapsed
  at genus level and the article explicitly did not compare against full-length sequencing.
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: FastQC
  role: Quality assessment of raw reads using the Q30 criterion and tabulating the
    number of reads per sample
- name: Deblur
  role: Denoising paired-end sequences into ASVs, deleting non-redundant reads
- name: Greengenes
  role: Taxonomic reference database; assignments collapsed at the genus level to
    produce taxonomy counts
- name: Illumina MiSeq
  role: Sequencing platform producing the paired-end reads (300 bp limit constrains
    profiling)
- name: QIAseq 16S/ITS screening panel (Qiagen)
  role: Library preparation for the Illumina platform prior to sequencing
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:73bacfda380529cdc1d1289bc5d074a2eae00dacc99b541bd245fbf4a06cb7ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - FastQC
  - Greengenes
  - Illumina MiSeq
  - QIAseq 16S/ITS screening panel (Qiagen)
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# microbiome-sequence-count-tabulation

## Summary

A reproducible workflow for tabulating post-QC sequence depth, ASV counts, and collapsed taxonomy counts from 16S rRNA amplicon sequencing of sputum samples across hypervariable region combinations (V1–V2, V3–V4, V5–V7, V7–V9). It standardizes the numbers a future agent needs to sanity-check a microbiome preprocessing run.

## When to use

Use when raw 16S rRNA amplicon FASTQ files (e.g., 33 sputum samples sequenced on Illumina MiSeq) must be quality-filtered, denoised, and reduced to per-sample sequence counts, per-region ASV totals, and genus-level taxonomy counts.

## When NOT to use

- Input is already a denoised feature/ASV table or a collapsed genus-level abundance table — this skill regenerates counts from raw FASTQ, so it would duplicate completed preprocessing.
- The data are full-length 16S rRNA reads from third-generation platforms (e.g., Nanopore) — the pipeline is tuned to Illumina MiSeq's 300 bp length limit and paired-end Deblur.
- The goal is functional profiling or strain-level resolution — Greengenes is collapsed at genus level and the article explicitly did not compare against full-length sequencing.

## Inputs

- Raw demultiplexed paired-end FASTQ files from 33 human sputum samples (Illumina MiSeq, QIAseq 16S/ITS screening panel)
- QIIME 2 non-chimeric 16S rRNA reference database for chimera checking
- Greengenes taxonomy database (for genus-level collapsing)

## Outputs

- Per-sample read/sequence counts after Q30 filtering (target ~100,000 sequences/sample)
- ASV table per hypervariable region combination (V1–V2, V3–V4, V5–V7, V7–V9; reported total 1,568,227 ASVs per region)
- Count of distinct taxonomies at genus level (reported: 114)
- Machine-readable preprocessing_summary.csv aggregating the above

## How to apply

Assess raw reads with FastQC (Version 0.11.9) applying the Q30 quality criterion and tabulate the number of reads per sample (the reference run averaged ~100,000 sequences per sample over 33 sputum samples). Denoise the paired-end data with the Deblur pipeline (as implemented in QIIME 2), deleting non-redundant reads, then flag chimeras against the QIIME 2 non-chimeric 16S rRNA database using a best-hit similarity threshold below 99% to obtain the ASV table (the study reported 1,568,227 ASVs per hypervariable region). Assign taxonomy with the Greengenes database collapsed at the genus level (QIIME 2) and count distinct taxonomies per region combination (114 in the reference study). Finally, aggregate per-sample sequence counts, per-region ASV totals, and taxonomy counts into a machine-readable table (e.g., preprocessing_summary.csv) so the reproduced figures can be diffed against the published summary.

## Related tools

- **FastQC** (Quality assessment of raw reads using the Q30 criterion and tabulating the number of reads per sample)
- **Deblur** (Denoising paired-end sequences into ASVs, deleting non-redundant reads)
- **Greengenes** (Taxonomic reference database; assignments collapsed at the genus level to produce taxonomy counts)
- **Illumina MiSeq** (Sequencing platform producing the paired-end reads (300 bp limit constrains profiling))
- **QIAseq 16S/ITS screening panel (Qiagen)** (Library preparation for the Illumina platform prior to sequencing)

## Examples

```
fastqc raw_reads/*.fastq.gz --outdir fastqc_out --threads 4  # Q30 read counts per sample; then Deblur denoising and Greengenes genus collapsing in QIIME 2
```

## Evaluation signals

- Per-sample depth after Q30 filtering should approximate ~100,000 sequences per sample across the 33 sputum samples.
- Total ASV count per hypervariable region combination should approach 1,568,227; flag any region whose count deviates substantially from the reported figure.
- Distinct genus-level taxonomy count should match 114 for the reference study.
- Chimera filtering must reference the QIIME 2 non-chimeric 16S rRNA database with a best-hit similarity threshold below 99%.
- Final output must be a machine-readable table (preprocessing_summary.csv) whose values match the published summary within expected tolerance.

## Limitations

- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling and thus the ASV/depth counts this skill produces.
- Taxonomy was trained with Greengenes despite more updated versions of SILVA being available, which may bias genus-level counts.
- The ZymoBIOMICS mock control contains only 10 species, affecting the sensitivity and specificity of taxonomic classification and validation.
- No comparison to full-length 16S rRNA sequencing was performed, so counts cannot be benchmarked against full-length data.

## Evidence

- [methods] FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011: "FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011"
- [methods] Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [methods] The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [readme] Hypervariable regions of 16S rRNA with the highest resolving power for taxonomic identification: "Hypervariable regions of 16S rRNA with the highest resolving power for taxonomic identification"
