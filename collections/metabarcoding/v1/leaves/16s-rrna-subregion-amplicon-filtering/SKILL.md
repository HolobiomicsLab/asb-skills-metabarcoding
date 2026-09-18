---
name: 16s-rrna-subregion-amplicon-filtering
description: Use when full-length 16S reference sequences must be converted into in-silico
  amplicons for commonly used sub-regions (e.g., V4, V1–V3, V3–V5) to assess how much
  taxonomic resolution short-read sub-region sequencing loses relative to the full
  gene.
when_to_use_negative:
- Input sequences are already primer-trimmed amplicon reads or a feature table rather
  than full-length 16S reference sequences with primer-binding sites — Cutadapt will
  find no amplicon and everything will be discarded.
- The goal is to exploit the full discriminatory potential of the ~1500 bp gene (e.g.,
  full-length Nanopore/PacBio analysis); this skill deliberately restricts sequences
  to partial variable regions.
- Reference sequences contain ambiguous 'N' bases or lack primer-binding sites for
  the chosen region — these are excluded by design, so the skill would silently reduce
  database representation.
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: Cutadapt
  role: Excises the sub-region amplicon spanning the targeted variable region from
    each full-length Greengenes 16S sequence, matching PCR primers with a maximum
    of 3 mismatches
- name: ruffus
  role: CGAT ruffus pipeline framework in which the study's figure-level Python pipelines
    (including this filtering workflow) are organised and run
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:eb8a235722b0b1fbbb75a533af0a4f03f4e8441fd146ca58347b9acf533ace9e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cutadapt
  - ruffus
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# 16s-rrna-subregion-amplicon-filtering

## Summary

In-silico trimming of full-length 16S rRNA reference sequences (non-redundant Greengenes v13.8.99 FASTA) into sub-region amplicons (V4, V1–V3, V3–V5, V6–V9, V1–V9) using Cutadapt with study PCR primer pairs at ≤3 mismatches, then no-'N' and ±2-SD length filtering, to compare taxonomic resolution of sub-regions vs. the full ~1500 bp gene.

## When to use

Use when full-length 16S reference sequences must be converted into in-silico amplicons for commonly used sub-regions (e.g., V4, V1–V3, V3–V5) to assess how much taxonomic resolution short-read sub-region sequencing loses relative to the full gene.

## When NOT to use

- Input sequences are already primer-trimmed amplicon reads or a feature table rather than full-length 16S reference sequences with primer-binding sites — Cutadapt will find no amplicon and everything will be discarded.
- The goal is to exploit the full discriminatory potential of the ~1500 bp gene (e.g., full-length Nanopore/PacBio analysis); this skill deliberately restricts sequences to partial variable regions.
- Reference sequences contain ambiguous 'N' bases or lack primer-binding sites for the chosen region — these are excluded by design, so the skill would silently reduce database representation.

## Inputs

- Non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA (from the study repository)
- Primer pairs for the five sub-regions (V4, V1–V3, V3–V5, V6–V9, V1–V9) from Supplementary Table 1

## Outputs

- Five filtered FASTA files, one per sub-region: V4, V1-V3, V3-V5, V6-V9, V1-V9
- Per-sub-region mean and standard deviation of in-silico amplicon lengths (used for the ±2-SD length filter)

## How to apply

Load the non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA from the study repository and retrieve the primer pairs for each of the five sub-regions (V4, V1–V3, V3–V5, V6–V9, V1–V9) from Supplementary Table 1. For each primer set, run Cutadapt allowing a maximum of 3 mismatches to excise the amplicon spanning the targeted variable region from every database sequence; discard extracted sequences where either primer fails to find an amplicon and sequences containing ambiguous 'N' bases. Compute the mean and standard deviation of amplicon lengths per sub-region and discard sequences longer or shorter than 2 standard deviations from the regional mean. Write the remaining sequences to one filtered FASTA file per sub-region. The rationale is to emulate, in silico, the sequence space a short-read sub-region assay would actually observe, so that downstream taxonomic-accuracy comparisons against the full ~1500 bp gene are fair.

## Related tools

- **Cutadapt** (Excises the sub-region amplicon spanning the targeted variable region from each full-length Greengenes 16S sequence, matching PCR primers with a maximum of 3 mismatches)
- **ruffus** (CGAT ruffus pipeline framework in which the study's figure-level Python pipelines (including this filtering workflow) are organised and run) — https://github.com/cgat-developers/ruffus

## Evaluation signals

- Exactly five filtered FASTA files are produced, one per sub-region: V4, V1-V3, V3-V5, V6-V9, and V1-V9.
- Every retained sequence passed a 3-mismatch primer match for both primers of its sub-region (no amplicon found by either primer means the sequence was discarded).
- No retained sequence contains ambiguous 'N' bases.
- All retained amplicon lengths fall within 2 standard deviations of the regional mean length.
- Mean and standard deviation of amplicon length were computed per sub-region before applying the length filter.

## Limitations

- None of the variable regions covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene — the in-silico amplicons are inherently lower-resolution proxies.
- Sequences whose variable region is missing (no amplicon found by either primer) or containing 'N' bases are discarded, so the filtered databases are subsets of the original Greengenes database.
- The per-region mean-length ±2-SD cutoff assumes approximately normal amplicon-length distributions; unusually long or short variants are removed by design.

## Evidence

- [intro] in-silico sub-region motivation: "Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [abstract] sub-region vs full gene: "We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene."
- [readme] pipeline framework: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures. Each pipeline is designed to be run using a [CGAT ruffus framework]"
- [discussion] no region recaptures full-gene diversity: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
