---
name: sample-depth-filtering-and-rarefaction
description: Use when preparing per-study Deblur or OTU BIOM tables for cross-study
  meta-analysis and you need equal sequencing depth — e.g., before merging EMP 16S
  Release 1 tables or comparing alpha/beta diversity across studies.
when_to_use_negative:
- The input table is already rarefied or you plan compositionally-aware methods (e.g.,
  differential abundance tests) where rarefying discards valid low-depth samples below
  5,000 sequences
- The per-study table contains clustered OTUs rather than exact Deblur sequences,
  since cross-study tracking of the 90-bp V4 tag sequences depends on exact-sequence
  features
- Samples have fewer than 5,000 total reads, which cannot meet the rarefaction depth
  and would be dropped
edam_topics:
- http://edamontology.org/topic_0176
tools:
- name: Deblur
  role: Exact-sequence de novo OTU picking method used to generate the 90-bp V4 tag-sequence
    feature table that is then depth-filtered and rarefied
  repo: https://github.com/cuttlefishh/deblur
- name: QIIME1
  role: Used to merge the per-study rarefied BIOM tables and mapping files into the
    single QC-filtered table
- name: QIIME2
  role: Alternative to QIIME1 for merging the BIOM tables and mapping files
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:7f9b709072f986013b85a44cc1bd2d423ddcaa2107b2a16353f205a58769c44f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Python
  - QIIME1
  - QIIME2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0176
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Sample depth filtering and rarefaction

## Summary

Depth-based QC and normalization for EMP-style 16S feature tables: filter OTUs/sequences with the mc2 rule (minimum sequence count of 2), drop samples below the read threshold, then rarefy to a fixed depth (5,000 observations per sample) so cross-study comparisons are not driven by sequencing depth.

## When to use

Use when preparing per-study Deblur or OTU BIOM tables for cross-study meta-analysis and you need equal sequencing depth — e.g., before merging EMP 16S Release 1 tables or comparing alpha/beta diversity across studies.

## When NOT to use

- The input table is already rarefied or you plan compositionally-aware methods (e.g., differential abundance tests) where rarefying discards valid low-depth samples below 5,000 sequences
- The per-study table contains clustered OTUs rather than exact Deblur sequences, since cross-study tracking of the 90-bp V4 tag sequences depends on exact-sequence features
- Samples have fewer than 5,000 total reads, which cannot meet the rarefaction depth and would be dropped

## Inputs

- Per-study Deblur-processed BIOM feature table (90-bp V4 tag sequences)
- Per-study demultiplexed fastq-derived 'demux' tables from the EMP 16S Release 1 FTP deposit (ftp://ftp.microbio.me/emp/release1)
- Mapping files for the samples included in each per-study table

## Outputs

- Rarefied per-study BIOM table at 5,000 observations per sample
- Merged QC-filtered Deblur 90-bp BIOM table (emp_deblur_90bp_qc.biom) spanning 96 studies and 23,828 samples

## How to apply

Start from per-study demultiplexed ('demux') tables that have been processed with Deblur and trimmed to 90 bp to match the V4 tag length. First apply the mc2 filter, which keeps only OTUs/sequences with a minimum total count of 2, then retain only samples with >=25 total reads. Rarefy the filtered feature table to exactly 5,000 observations per sample — samples that cannot reach this depth are excluded. Finally, merge the per-study BIOM tables with QIIME1 or QIIME2 into a single table; for EMP 16S Release 1 this yields the QC-filtered Deblur 90-bp table with 307,572 unique sequences across 96 studies and 23,828 samples. The rationale is that exact-sequence (Deblur) features can be followed across studies only if depth is harmonized, since alpha/beta diversity metrics are depth-sensitive.

## Related tools

- **Deblur** (Exact-sequence de novo OTU picking method used to generate the 90-bp V4 tag-sequence feature table that is then depth-filtered and rarefied) — https://github.com/cuttlefishh/deblur
- **QIIME1** (Used to merge the per-study rarefied BIOM tables and mapping files into the single QC-filtered table)
- **QIIME2** (Alternative to QIIME1 for merging the BIOM tables and mapping files)

## Evaluation signals

- Every retained sample's total observation count equals exactly 5,000 after rarefaction
- Merged QC-filtered table reproduces the published dimensions: 96 studies, 23,828 samples, and 307,572 unique sequences
- No OTU/sequence in the filtered table has a total count below 2 (mc2 rule satisfied)
- All retained samples have >=25 total reads prior to rarefaction
- Output is a valid BIOM-format file (e.g., emp_deblur_90bp_qc.biom) loadable by QIIME1/QIIME2

## Limitations

- Rarefying to 5,000 sequences per sample discards samples with lower sequencing depth, reducing sample coverage relative to the full ~25,000-sample/97-study EMP 16S Release 1 set
- The published table trims to 90 bp matching the V4 region starting after primer 515f; sequences from other variable regions or lengths are not compatible
- Rarefaction discards reads beyond the 5,000-observation cap, so depth-sensitive abundance information is not retained

## Evidence

- [supplementary] Rarefaction depth: "Samples were rarefied to 5,000 observations per sample."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [methods] Table dimensions after processing: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [abstract] Rationale for exact sequences: "Coordinated protocols and new analytical methods, particularly the use of exact sequences instead of clustered operational taxonomic units, enable bacterial and archaeal ribosomal RNA gene sequences"
