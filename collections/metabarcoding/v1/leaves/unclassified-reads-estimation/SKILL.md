---
name: unclassified-reads-estimation
description: Use when profiling shotgun metagenomic samples with MetaPhlAn 4 and you
  need to quantify the fraction of reads explained (including reads mapping to uSGBs),
  or when comparing MetaPhlAn 4 vs MetaPhlAn 3 explainable-read fractions across environments.
when_to_use_negative:
- The goal is functional profiling (genes/pathways) rather than taxonomic profiling
  — MetaPhlAn is a taxonomic profiler; use HUMAnN for functional profiles.
- The sample comes from environmental, nonhost-associated, or under-studied microbial
  communities where sequences remain highly enriched for those not captured even by
  current uSGBs — explainable-read gains will be limited and interpretation should
  be cautious.
- Input is already a species-level relative-abundance table from another profiler
  and no re-profiling with MetaPhlAn 4 is needed.
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: MetaPhlAn 4
  role: Primary profiler run with unclassified-reads estimation enabled, using the
    uSGB-extended marker database to explain reads from uncharacterized species
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Baseline profiler (v3.0.12) whose explainable-read fractions serve as the
    reference for computing fold increases
  repo: https://github.com/biobakery/MetaPhlAn
- name: PhyloPhlAn
  role: Provides phylogenetic characterization workflows for the SGBs underlying the
    marker database
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:f172900a6b4a22c25f801f49ba37b6e71abdf6683a9c311b4e223f5897e89b43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - PhyloPhlAn
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# unclassified-reads-estimation

## Summary

Runs MetaPhlAn 4 with its unclassified-reads estimation feature enabled so that shotgun metagenomic reads assigned to taxonomically unidentified species-level genome bins (uSGBs) are counted as 'explainable', quantifying how much more of each sample MetaPhlAn 4 profiles versus MetaPhlAn 3.

## When to use

Use when profiling shotgun metagenomic samples with MetaPhlAn 4 and you need to quantify the fraction of reads explained (including reads mapping to uSGBs), or when comparing MetaPhlAn 4 vs MetaPhlAn 3 explainable-read fractions across environments.

## When NOT to use

- The goal is functional profiling (genes/pathways) rather than taxonomic profiling — MetaPhlAn is a taxonomic profiler; use HUMAnN for functional profiles.
- The sample comes from environmental, nonhost-associated, or under-studied microbial communities where sequences remain highly enriched for those not captured even by current uSGBs — explainable-read gains will be limited and interpretation should be cautious.
- Input is already a species-level relative-abundance table from another profiler and no re-profiling with MetaPhlAn 4 is needed.

## Inputs

- Shotgun metagenomic FASTQ files (e.g. from the 24,515-sample / 145 public dataset collection)
- MetaPhlAn 4 marker database including 4,992 taxonomically unidentified uSGBs
- MetaPhlAn 3.0.12 baseline profiles for the same samples

## Outputs

- Per-sample species-level relative-abundance profiles with unclassified/read-explainability estimates (MetaPhlAn profile files)
- Per-environment fold-increase in explainable reads of MetaPhlAn 4 vs MetaPhlAn 3.0.12 (e.g. environment_read_explainability_summary.tsv)
- Per-sample fraction of profiled richness attributed to uSGBs

## How to apply

Run MetaPhlAn 4 (marker database of 26,970 species-level genome bins from 1.01 M prokaryotic reference and metagenome-assembled genomes, including 4,992 taxonomically unidentified uSGBs) on each FASTQ sample with the unclassified-reads estimation feature enabled, and run MetaPhlAn 3.0.12 on the same samples as baseline to obtain species-level relative-abundance profiles and explainable-read fractions. Group samples by host environment (e.g. human gut, airways, wild mice, rumen, ocean) and compute, per environment, the fold increase in explainable reads of MetaPhlAn 4 relative to MetaPhlAn 3.0.12 (reported anchors: ~20% more reads in human gut, 1.95-fold airways, 3.26-fold wild mice, 14.15-fold rumen, 6.65-fold ocean). Also compute the per-sample fraction of profiled richness attributed to uSGBs, summarized across samples (mean 23.13%, s.d. 17.89%). The rationale is that integrating metagenome-assembled genome markers lets MetaPhlAn 4 quantify organisms with no cultured isolates, which is what 'explains' reads previously left unclassified.

## Related tools

- **MetaPhlAn 4** (Primary profiler run with unclassified-reads estimation enabled, using the uSGB-extended marker database to explain reads from uncharacterized species) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Baseline profiler (v3.0.12) whose explainable-read fractions serve as the reference for computing fold increases) — https://github.com/biobakery/MetaPhlAn
- **PhyloPhlAn** (Provides phylogenetic characterization workflows for the SGBs underlying the marker database)

## Examples

```
conda install -c bioconda metaphlan && metaphlan sample.fastq.gz --input_type fastq --unclassified_estimation -o sample_profile.tsv
```

## Evaluation signals

- Rumen samples reproduce the reported 14.15-fold increase in explainable reads over MetaPhlAn 3.0.12
- Ocean samples reproduce the reported 6.65-fold increase; airways 1.95-fold; wild mice 3.26-fold
- Human gut samples show ~20% more explained reads than MetaPhlAn 3
- Mean per-sample fraction of profiled richness attributed to uSGBs matches 23.13% (s.d. 17.89%)
- MetaPhlAn 4 profiles remain species-level and more accurate than alternatives on synthetic evaluation communities while quantifying organisms without cultured isolates

## Limitations

- Even with the extended SGB and marker set, further work remains to better profile under-characterized habitats
- Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] ~20% more reads explained in human gut: "MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome"
- [discussion] Strategy integrating assembly with reference-based profiling: "MetaPhlAn 4 provides a strategy for integrating metagenomic assembly with reference-based profiling approaches to achieve novelty"
- [readme] Installation via conda (readme): "To install MetaPhlan through conda run:

``n$ conda install -c bioconda metaphlan
"
- [intro] Tool scope: species-level profiling from shotgun data: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
