---
name: asv-feature-table-generation
description: Use when raw paired-end 16S amplicon FASTQ reads (e.g., 2 x 300 bp Illumina
  MiSeq libraries such as V1–V2, V3–V4, V5–V7, V7–V9) must be denoised into ASVs as
  an alternative to OTU binning.
when_to_use_negative:
- Input is already a denoised ASV feature table (e.g., BIOM) or the goal is downstream
  diversity/taxonomy analysis rather than read denoising.
- 'Reads are full-length 16S rRNA sequences from third-generation platforms: the article
  notes Deblur is recommended for shorter bp fragments and that DADA2 identified more
  real variants, so it is the better choice there.'
- The aim is strain- or species-level resolution beyond collapsed genus-level assignments;
  this skill ends at genus-level taxonomic collapsing via Greengenes.
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: Deblur
  role: Denoises Q30-filtered paired-end 16S reads into ASVs and deletes non-redundant
    reads, using a static error model instead of OTU binning
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: FastQC
  role: 'Pre-denoising quality assessment: obtains Q30 quality sequences and inspects
    reads per sample, bases per sequence, and sequence length distribution'
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: QIIME 2
  role: Provides the non-chimeric 16S rRNA database used to identify chimeras among
    denoised reads, removing best hits with similarity < 99%
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:ea76757cb13412cce0499e170d7f40f6f3508e71601b55f9020120becf51a4b2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - FastQC
  - QIIME 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# asv-feature-table-generation

## Summary

Generates bacterial amplicon sequence variants (ASVs) from 16S amplicon reads by quality-filtering with FastQC (Q30), denoising with the Deblur pipeline, and screening chimeras against QIIME 2's non-chimeric 16S rRNA database, producing a BIOM feature table for genus-level assignment.

## When to use

Use when raw paired-end 16S amplicon FASTQ reads (e.g., 2 x 300 bp Illumina MiSeq libraries such as V1–V2, V3–V4, V5–V7, V7–V9) must be denoised into ASVs as an alternative to OTU binning.

## When NOT to use

- Input is already a denoised ASV feature table (e.g., BIOM) or the goal is downstream diversity/taxonomy analysis rather than read denoising.
- Reads are full-length 16S rRNA sequences from third-generation platforms: the article notes Deblur is recommended for shorter bp fragments and that DADA2 identified more real variants, so it is the better choice there.
- The aim is strain- or species-level resolution beyond collapsed genus-level assignments; this skill ends at genus-level taxonomic collapsing via Greengenes.

## Inputs

- Paired-end 16S amplicon FASTQ reads (e.g., 2 x 300 bp Illumina MiSeq from V1–V2, V3–V4, V5–V7, V7–V9 library pools)
- QIIME 2 non-chimeric 16S rRNA reference database for chimera screening

## Outputs

- Denoised ASV feature table (BIOM format)
- Representative ASV sequences for downstream genus-level taxonomic assignment

## How to apply

First assess per-base quality and reads per sample with FastQC (Version 0.11.9 in the article) and filter raw reads to Q30. Then run the Deblur denoising pipeline, which is intended for paired-end data, to infer ASVs while deleting non-redundant reads; Deblur is recommended for shorter bp fragments such as short Illumina 16S amplicons. Screen the denoised reads against the non-chimeric 16S rRNA database from QIIME 2 and remove reads whose best hit similarity rate is less than 99% (i.e., candidate chimeras). Export the denoised ASV feature table and representative sequences as inputs for downstream taxonomic assignment, which the article obtained using the Greengenes database and collapsed at the genus level.

## Related tools

- **Deblur** (Denoises Q30-filtered paired-end 16S reads into ASVs and deletes non-redundant reads, using a static error model instead of OTU binning) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **FastQC** (Pre-denoising quality assessment: obtains Q30 quality sequences and inspects reads per sample, bases per sequence, and sequence length distribution) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **QIIME 2** (Provides the non-chimeric 16S rRNA database used to identify chimeras among denoised reads, removing best hits with similarity < 99%) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification

## Evaluation signals

- All raw reads pass a Q30 quality cut-off confirmed in FastQC (reads per sample, bases per sequence, and sequence length distribution inspected).
- Non-redundant reads are deleted during Deblur denoising; the denoised output contains inferred ASVs, not OTUs.
- Chimeric reads are removed: any read with a best hit similarity rate below 99% against the QIIME 2 non-chimeric 16S rRNA database is discarded.
- The final feature table is exported in BIOM format together with representative ASV sequences suitable for genus-level Greengenes assignment.
- Denoised ASV tables are produced separately for each hypervariable-region combination (V1–V2, V3–V4, V5–V7, V7–V9) so they can be cross-validated against the ZymoBIOMICS mock community.

## Limitations

- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling and constrains the fragments suitable for Deblur.
- DADA2 identified more real variants and output sequences than other methods in the article's discussion, despite Deblur being recommended for shorter bp fragments.
- Taxonomic assignment relies on the Greengenes database even though more updated versions of SILVA were available, potentially biasing genus-level classifications.

## Evidence

- [abstract] We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
- [methods] Deblur pipeline on paired-end data: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [methods] Chimera screening threshold: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [methods] FastQC quality filtering: "FastQC Version 0.11.9 assessed quality and the filtering of raw reads with a cut-off score of up to 3011"
- [methods] Genus-level collapsing: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [results] FastQC usage in results: "FastQC was used to obtain quality sequences with Q30 and to observe the number of reads per sample, bases per sequence, sequence distribution length, and scores per sequence"
- [discussion] Deblur suitability for short fragments: "despite Deblur being recommended for shorter bp fragments"
