---
name: chimera-detection-in-16s-reads
description: Use when raw 16S amplicon FASTQ reads (paired-end Illumina MiSeq, e.g.
  V1–V2/V3–V4/V5–V7/V7–V9 pools) have been quality-filtered to Q30 and denoised with
  Deblur, and chimeric artifacts must be removed before genus-level taxonomic collapsing.
when_to_use_negative:
- Input is not 16S rRNA amplicon sequence data (e.g. shotgun metagenomic or ITS reads)
  — the QIIME 2 non-chimeric 16S rRNA reference and 99% threshold are calibrated for
  16S amplicons
- Reads have not yet been quality-filtered (Q30) and denoised with Deblur — chimera
  screening is a downstream filter step, not a replacement for denoising
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Provides the non-chimeric 16S rRNA database used as the reference for identifying
    chimeras in the denoised reads
- name: Deblur
  role: Upstream denoising algorithm (for paired-end data) that produces the non-redundant
    denoised reads entering chimera screening
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: FastQC
  role: Upstream quality assessment of raw reads (Q30) prior to denoising and chimera
    filtering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:2545877af4a456857d36c74b05cc160a2c6e3284e63c0b9000c16dd62ca37801
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Chimera detection in 16S reads

## Summary

Screens denoised 16S rRNA amplicon reads against the non-chimeric 16S rRNA database from QIIME 2, removing reads whose best hit similarity rate is less than 99%, so that downstream ASV-level (genus) taxonomic assignment with Deblur is not corrupted by chimeric sequences.

## When to use

Use when raw 16S amplicon FASTQ reads (paired-end Illumina MiSeq, e.g. V1–V2/V3–V4/V5–V7/V7–V9 pools) have been quality-filtered to Q30 and denoised with Deblur, and chimeric artifacts must be removed before genus-level taxonomic collapsing.

## When NOT to use

- Input is not 16S rRNA amplicon sequence data (e.g. shotgun metagenomic or ITS reads) — the QIIME 2 non-chimeric 16S rRNA reference and 99% threshold are calibrated for 16S amplicons
- Reads have not yet been quality-filtered (Q30) and denoised with Deblur — chimera screening is a downstream filter step, not a replacement for denoising

## Inputs

- Denoised 16S rRNA amplicon reads (output of the Deblur pipeline on paired-end MiSeq data)
- Non-chimeric 16S rRNA reference database from QIIME 2

## Outputs

- Chimera-filtered denoised reads (reads with best hit similarity >= 99%)
- Set of removed chimeric reads (best hit similarity < 99%)

## How to apply

After quality assessment with FastQC (Q30 cut-off) and Deblur denoising — which is intended for paired-end data and deletes non-redundant reads — take the denoised reads and screen them against the non-chimeric 16S rRNA database from QIIME 2. Identify chimeras by similarity search: any read whose best hit similarity rate is less than 99% is treated as chimeric and removed. The decision point is the 99% best-hit threshold; reads at or above this similarity against the non-chimeric reference set are retained. Rationale: chimeras generated during PCR of short hypervariable-region amplicons would otherwise appear as spurious ASVs and distort genus-level relative abundances and downstream diversity statistics (Kruskal–Wallis alpha diversity, NMDS of Bray–Curtis dissimilarities).

## Related tools

- **QIIME 2** (Provides the non-chimeric 16S rRNA database used as the reference for identifying chimeras in the denoised reads)
- **Deblur** (Upstream denoising algorithm (for paired-end data) that produces the non-redundant denoised reads entering chimera screening) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **FastQC** (Upstream quality assessment of raw reads (Q30) prior to denoising and chimera filtering)

## Evaluation signals

- Retained reads show a best hit similarity rate of at least 99% against the QIIME 2 non-chimeric 16S rRNA database
- Every read with a best hit similarity rate below 99% is removed from the denoised set
- Chimera filtering is applied only after Deblur denoising (which deletes non-redundant reads) and before Greengenes-based genus-level taxonomic collapsing
- The number of retained reads remains consistent with per-sample read counts observed in FastQC-based quality summaries

## Limitations

- The 99% best-hit threshold was applied against Greengenes-era QIIME 2 reference data, and the article notes the regions were trained with Greengenes despite more updated SILVA versions being available
- The 300 bp length limit of the Illumina MiSeq platform constrains 16S rRNA profiling and may affect chimera discrimination across region combinations (V1–V2, V3–V4, V5–V7, V7–V9)
- Denoising and chimera screening were optimized for short fragments — Deblur is recommended for shorter bp fragments — so behavior on long or full-length reads is not established by this study

## Evidence

- [methods] Chimera identification: "The non-chimeric 16S rRNA database from QIIME 2 was used to identify chimeras in the reads, with a best hit similarity rate of less than 99%"
- [methods] Sequence denoising with Deblur: "Sequence denoising was performed with the Deblur pipeline, which is intended for paired-end data, and the non-redundant reads were deleted"
- [abstract] ASV identification: "We used the Deblur algorithm to identify bacterial amplicon sequence variants (ASVs) at the genus level"
