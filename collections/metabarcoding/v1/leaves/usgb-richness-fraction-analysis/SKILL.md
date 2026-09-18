---
name: usgb-richness-fraction-analysis
description: Use when comparing MetaPhlAn 4 vs MetaPhlAn 3.0.12 read-explainability
  across environments (human gut, airways, wild mice, rumen, ocean) or estimating
  the uSGB-attributed fraction of profiled richness per sample.
when_to_use_negative:
- Input is functional profiling data (HUMAnN-style pathway/gene tables) rather than
  taxonomic read-explainability analysis
- Samples are viral or eukaryote-dominated viromes/metatranscriptomes — the current
  MetaPhlAn 4 methods 'do not extensively incorporate viral or eukaryotic microbial
  sequences'
- Only a handful of samples from a single well-characterized environment, where fold-increase
  and uSGB-richness statistics across environments are not meaningful
edam_operation: http://edamontology.org/operation_0269
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaPhlAn 4
  role: Profiler run with unclassified-reads estimation to obtain SGB/uSGB-level profiles
    and explainable-read fractions
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn 3.0.12
  role: Baseline profiler for computing per-environment fold increases in explainable
    reads
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn (bioconda)
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:59e427741ffb4d22f10de092ef74df176a379fa0d2dd2cca2deada3abfa40df0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 3.0.12
  - MetaPhlAn 4
  - conda
  edam_operation: http://edamontology.org/operation_0269
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# usgb-richness-fraction-analysis

## Summary

Quantify how much of the per-sample profiled microbial richness is attributed to taxonomically unidentified species-level genome bins (uSGBs) when profiling metagenomes with MetaPhlAn 4, and compute environment-specific fold increases in explainable reads relative to MetaPhlAn 3.

## When to use

Use when comparing MetaPhlAn 4 vs MetaPhlAn 3.0.12 read-explainability across environments (human gut, airways, wild mice, rumen, ocean) or estimating the uSGB-attributed fraction of profiled richness per sample.

## When NOT to use

- Input is functional profiling data (HUMAnN-style pathway/gene tables) rather than taxonomic read-explainability analysis
- Samples are viral or eukaryote-dominated viromes/metatranscriptomes — the current MetaPhlAn 4 methods 'do not extensively incorporate viral or eukaryotic microbial sequences'
- Only a handful of samples from a single well-characterized environment, where fold-increase and uSGB-richness statistics across environments are not meaningful

## Inputs

- Shotgun metagenomic samples (e.g. FASTQ reads) from the 24,515-sample collection spanning 145 public datasets
- MetaPhlAn 4 marker database (26,970 SGBs incl. 4,992 uSGBs)
- MetaPhlAn 3.0.12 reference database for baseline profiling
- Environment/host labels per sample (human gut, airways, wild mice, rumen, ocean, etc.)

## Outputs

- Per-environment fold increases in explainable reads (MetaPhlAn 4 vs MetaPhlAn 3.0.12)
- Per-sample fraction of profiled richness attributed to uSGBs (mean 23.13%, s.d. 17.89% expected)
- environment_read_explainability_summary.tsv

## How to apply

Organize the shotgun metagenomic samples by host environment and run MetaPhlAn 3.0.12 on each to obtain baseline species-level relative-abundance profiles and explainable-read fractions. Then run MetaPhlAn 4 with its unclassified-reads estimation feature enabled, leveraging the marker database of 26,970 species-level genome bins derived from 1.01 M prokaryotic reference and metagenome-assembled genomes, of which 4,992 are taxonomically unidentified uSGBs. For each environment, compute the fold increase in explainable reads of MetaPhlAn 4 relative to MetaPhlAn 3.0.12 (expected anchors: ~20% human gut, 1.95-fold airways, 3.26-fold wild mice, 14.15-fold rumen, 6.65-fold ocean). Next, compute the per-sample fraction of profiled richness attributed to uSGBs and summarize across samples, expecting a mean of 23.13% (s.d. 17.89%). Write the per-environment fold increases and uSGB richness fractions to a summary TSV (environment_read_explainability_summary.tsv) and check values against the reported anchors.

## Related tools

- **MetaPhlAn 4** (Profiler run with unclassified-reads estimation to obtain SGB/uSGB-level profiles and explainable-read fractions) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn 3.0.12** (Baseline profiler for computing per-environment fold increases in explainable reads) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn (bioconda)) — https://github.com/biobakery/MetaPhlAn

## Examples

```
metaphlan sample.fastq.bz2 --input_type fastq --bowtie2db metaphlan_databases/mpa_vJan21_CHOCOPhlAnSGB_202103 --unclassified_estimation > sgb_profile.tsv
```

## Evaluation signals

- Rumen fold increase reproduces 14.15-fold and ocean reproduces 6.65-fold
- Airways fold increase matches 1.95-fold and wild mice matches 3.26-fold
- Human gut read-explainability increase is approximately ~20%
- Per-sample uSGB richness fraction mean equals 23.13% with s.d. 17.89%
- Output TSV environment_read_explainability_summary.tsv contains one fold-increase row per environment and matches the reported anchors

## Limitations

- Further work remains to better profile under-characterized habitats
- Environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements

## Evidence

- [abstract] 26,970 SGB marker set including uSGBs: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] MetaPhlAn 4 explainable-read gains: "MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome"
- [intro] MetaPhlAn scope: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] under-characterized habitats limitation: "Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats."
- [readme] installation via bioconda: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
