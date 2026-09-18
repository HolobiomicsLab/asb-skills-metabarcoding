---
name: cross-environment-read-explainability-comparison
description: Use when comparing two profiler versions' fraction of explainable (classified)
  reads over the same metagenomic sample collection grouped by environment, e.g. benchmarking
  MetaPhlAn 4 vs MetaPhlAn 3.0.12 across >24,500 samples.
when_to_use_negative:
- The input is functional profiling data (e.g. HUMAnN UniRef90/UniRef50 annotations)
  rather than taxonomic read-explainability comparison
- Only one profiler version is being run — there is no baseline (e.g. MetaPhlAn 3.0.12)
  to compute a fold increase against
- The goal is strain-level population genomics (StrainPhlAn) rather than comparing
  classified-read fractions across environments
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaPhlAn 4
  role: Runs with unclassified-reads estimation enabled to obtain the treatment explainable-read
    fractions and uSGB-based richness using its 26,970-SGB marker database
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn 3.0.12
  role: Runs on each sample to provide the baseline species-level profiles and explainable-read
    fractions for the fold-increase comparison
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:b49571fe12ffdd44318ad885f0c43e69e4104b0c397e8eb5e8cec4eccf518b2d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 3.0.12
  - MetaPhlAn 4
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# cross-environment-read-explainability-comparison

## Summary

Quantifies how much more of the shotgun metagenomic read content a newer profiler (MetaPhlAn 4, with its 26,970 SGB / 4,992 uSGB marker database and unclassified-reads estimation) can explain relative to a previous version (MetaPhlAn 3.0.12) across host environments such as human gut, airways, wild mice, rumen, and ocean.

## When to use

Use when comparing two profiler versions' fraction of explainable (classified) reads over the same metagenomic sample collection grouped by environment, e.g. benchmarking MetaPhlAn 4 vs MetaPhlAn 3.0.12 across >24,500 samples.

## When NOT to use

- The input is functional profiling data (e.g. HUMAnN UniRef90/UniRef50 annotations) rather than taxonomic read-explainability comparison
- Only one profiler version is being run — there is no baseline (e.g. MetaPhlAn 3.0.12) to compute a fold increase against
- The goal is strain-level population genomics (StrainPhlAn) rather than comparing classified-read fractions across environments

## Inputs

- 24,515 shotgun metagenomic samples from 145 public datasets (FASTQ reads)
- Environment/host labels for each sample (human gut, airways, wild mice, rumen, ocean, etc.)
- MetaPhlAn 3.0.12 marker database
- MetaPhlAn 4 marker database of 26,970 species-level genome bins (4,992 uSGBs)

## Outputs

- environment_read_explainability_summary.tsv with per-environment fold increases in explainable reads
- Per-sample fraction of profiled richness attributed to uSGBs (mean 23.13%, s.d. 17.89%)
- Species-level relative-abundance profiles from MetaPhlAn 3.0.12 and MetaPhlAn 4

## How to apply

Retrieve the shotgun metagenomic samples (here 24,515 samples from 145 public datasets) and organize them by host environment (human gut, airways, wild mice, rumen, ocean). Run the baseline profiler (MetaPhlAn 3.0.12) on each sample to obtain species-level relative-abundance profiles and explainable-read fractions, then run MetaPhlAn 4 with its unclassified-reads estimation feature enabled, leveraging its marker database of 26,970 species-level genome bins (including 4,992 taxonomically unidentified uSGBs) built from 1.01 M prokaryotic reference and metagenome-assembled genomes. For each environment, compute the fold increase in explainable reads of MetaPhlAn 4 relative to MetaPhlAn 3.0.12. Additionally compute, per sample, the fraction of profiled richness attributed to uSGBs and summarize across samples. Write per-environment fold increases and uSGB richness fractions to a summary table (environment_read_explainability_summary.tsv) and validate against the reported anchors.

## Related tools

- **MetaPhlAn 4** (Runs with unclassified-reads estimation enabled to obtain the treatment explainable-read fractions and uSGB-based richness using its 26,970-SGB marker database) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn 3.0.12** (Runs on each sample to provide the baseline species-level profiles and explainable-read fractions for the fold-increase comparison) — https://github.com/biobakery/MetaPhlAn

## Examples

```
metaphlan sample.fastq --input_type fastq --index mpa_vOct22_CHOCOPhlAnSGB_202212 --unclassified_estimation --outputfile sample_metaphlan4_profile.tsv
```

## Evaluation signals

- Fold-increase values reproduce the reported anchors: ~20% more reads in human gut, 1.95-fold airways, 3.26-fold wild mice, 14.15-fold rumen, 6.65-fold ocean
- Per-sample uSGB richness fraction matches mean 23.13% with s.d. 17.89%
- The summary table environment_read_explainability_summary.tsv contains a fold-increase row for every environment group present in the 24,515-sample collection
- MetaPhlAn 4 marker database statistics confirm 26,970 species-level genome bins with 4,992 taxonomically unidentified uSGBs
- All 145 public datasets are represented in the per-environment aggregation

## Limitations

- Even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats; environmental, nonhost-associated, and under-studied communities are still highly enriched for sequences not captured by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences due to their unique genomic architectures and quality control requirements
- Larger gains are concentrated in less-characterized environments (e.g. >40% in the rumen); gains in well-characterized environments such as the human gut are smaller (~20% more reads)

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, 4,992 of which are taxonomically unidentified at the species level.: [evidence span withheld — anchor status: not_found]
- [results] Explainable-read fold-increase anchors across environments.: "MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome"
- [intro] MetaPhlAn is a computational tool for species-level microbial profiling from shotgun data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [abstract] Application to >24,500 metagenomes highlights previously undetected species as biomarkers.: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [discussion] Even with the extended MetaPhlAn 4 SGB and marker set, under-characterized habitats remain hard to profile.: "Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats."
