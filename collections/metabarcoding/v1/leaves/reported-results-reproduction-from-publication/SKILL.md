---
name: reported-results-reproduction-from-publication
description: Use when reproducing published sequencing-depth and ASV/taxonomy counts
  from a 16S rRNA amplicon study — e.g., validating ~100,000 reads/sample, 1,568,227
  ASVs per region, and 114 taxonomies from deposited FASTQ files.
when_to_use_negative:
- Raw FASTQ files or pipeline code are not deposited in a public repository, so the
  reported counts can only be transcribed from the paper rather than reproduced
- The goal is a new biological comparison (e.g., alpha-diversity or NMDS testing between
  regions) rather than validating the paper's reported read-depth and ASV/taxonomy
  tallies
- Input is already a denoised ASV/feature table — re-running the FastQC/Deblur chain
  would not validate the reported per-sample sequencing depth
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: FastQC
  role: Assesses raw-read quality and filters FASTQ reads at Q30, tabulating the reads-per-sample
    depth being reproduced (Version 0.11.9)
- name: Deblur
  role: Denoises the paired-end reads and deletes non-redundant reads to produce the
    per-region ASV counts
- name: QIIME 2
  role: Supplies the non-chimeric 16S rRNA database for chimera flagging (<99% best-hit
    similarity) and the Greengenes genus-level taxonomic assignment
- name: Greengenes
  role: Reference classifier whose genus-level collapsing defines the reproduced taxonomy
    count (114) per region
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:3fa397b86c4418d02920d277980beb59234cce459dfcf597144881c2b18fa951
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - FastQC
  - Greengenes
  - Illumina MiSeq
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# reported-results-reproduction-from-publication

## Summary

Reproduces a 16S rRNA amplicon study's reported preprocessing statistics — per-sample sequencing depth after Q30 filtering, ASV counts per hypervariable region, and genus-level taxonomy counts — by re-running FastQC, Deblur, and Greengenes-based assignment on the deposited FASTQ files and diffing the recomputed tallies against the published figures.

## When to use

Use when reproducing published sequencing-depth and ASV/taxonomy counts from a 16S rRNA amplicon study — e.g., validating ~100,000 reads/sample, 1,568,227 ASVs per region, and 114 taxonomies from deposited FASTQ files.

## When NOT to use

- Raw FASTQ files or pipeline code are not deposited in a public repository, so the reported counts can only be transcribed from the paper rather than reproduced
- The goal is a new biological comparison (e.g., alpha-diversity or NMDS testing between regions) rather than validating the paper's reported read-depth and ASV/taxonomy tallies
- Input is already a denoised ASV/feature table — re-running the FastQC/Deblur chain would not validate the reported per-sample sequencing depth

## Inputs

- Raw FASTQ read files for the 33 sputum samples (deposited at the study's GitHub repository)
- QIIME 2 non-chimeric 16S rRNA reference database for chimera checking
- Greengenes taxonomy database (collapsed at the genus level)
- Published target values to reproduce: ~100,000 sequences/sample, 1,568,227 ASVs per region, 114 taxonomies

## Outputs

- Per-sample read counts after FastQC Q30 filtering
- ASV counts per region combination (V1–V2, V3–V4, V5–V7, V7–V9) after Deblur denoising and chimera removal
- Count of distinct genus-level taxonomies after Greengenes collapsing
- Machine-readable preprocessing_summary.csv reconciling recomputed values with the reported figures

## How to apply

Fetch the raw FASTQ read files for the 33 sputum samples from the deposited GitHub repository, then run FastQC (Version 0.11.9) to obtain Q30 quality sequences and tabulate reads per sample. Denoise the paired-end data with the Deblur pipeline, deleting non-redundant reads, and flag chimeras against the QIIME 2 non-chimeric 16S rRNA database at a best-hit similarity rate below 99%. Assign taxonomy with the Greengenes database collapsed at the genus level (QIIME 2), counting distinct taxonomies and total ASVs per region combination (V1–V2, V3–V4, V5–V7, V7–V9). Aggregate the per-sample sequence counts (~100,000 sequences/sample), the per-region ASV total (1,568,227), and the taxonomy count (114) into a machine-readable table such as preprocessing_summary.csv. Treat the reproduction as validated only when the recomputed values match the reported figures, and flag any region whose counts deviate.

## Related tools

- **FastQC** (Assesses raw-read quality and filters FASTQ reads at Q30, tabulating the reads-per-sample depth being reproduced (Version 0.11.9))
- **Deblur** (Denoises the paired-end reads and deletes non-redundant reads to produce the per-region ASV counts)
- **QIIME 2** (Supplies the non-chimeric 16S rRNA database for chimera flagging (<99% best-hit similarity) and the Greengenes genus-level taxonomic assignment)
- **Greengenes** (Reference classifier whose genus-level collapsing defines the reproduced taxonomy count (114) per region)

## Evaluation signals

- Recomputed per-sample depth averages ~100,000 sequences across all 33 sputum samples after Q30 filtering
- Per-region ASV totals match the reported 1,568,227 ASVs per hypervariable region; deviations per region are explicitly flagged
- Distinct genus-level taxonomies after Greengenes collapsing equal 114
- preprocessing_summary.csv exists and is machine-readable, covering per-sample sequence counts, per-region ASV totals, and taxonomy counts
- Chimera flagging applied the <99% best-hit similarity criterion against the QIIME 2 non-chimeric 16S rRNA database, consistent with the paper's methods

## Limitations

- The deposited GitHub repository's README contains only a title and no pipeline documentation or commands, so reproduction depends entirely on the paper's methods text
- The Illumina MiSeq 300 bp read-length limit constrains 16S rRNA profiling, so ASV recovery is region-dependent and cannot match full-length sequencing
- Taxonomy counts depend on Greengenes even though more updated SILVA versions were available; switching classifier or database will not reproduce the reported 114 taxonomies
- The paper's discussion notes DADA2 identified more real variants and output sequences than other methods, so an alternative denoiser will not reproduce the Deblur-based counts

## Evidence

- [abstract] DNA were isolated from 33 human sputum samples: "DNA were isolated from 33 human sputum samples"
- [methods] Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [methods] The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [results] FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample, bases per sequence, sequence distribution length, and scores per sequence: "FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample, bases per sequence, sequence distribution length, and scores per sequence"
- [discussion] the 300 bp length limit of the Illumna MiSeq platform limits 16S rRNA profiling: "the 300 bp length limit of the Illumna MiSeq platform limits 16S rRNA profiling"
- [readme] # Hypervariable regions of 16S rRNA with the highest resolving power for taxonomic identification: "# Hypervariable regions of 16S rRNA with the highest resolving power for taxonomic identification"
