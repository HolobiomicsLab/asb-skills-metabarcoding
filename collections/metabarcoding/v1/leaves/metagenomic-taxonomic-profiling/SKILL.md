---
name: metagenomic-taxonomic-profiling
description: Use when shotgun metagenomic FASTQ samples need species-/SGB-level relative
  abundance profiles, especially when uncharacterized species (uSGBs), read-explainability
  gains, or prevalence estimates across large sample collections are the analysis
  goal.
when_to_use_negative:
- Input is already a taxonomic abundance/feature table — no re-profiling with marker
  genes is needed
- The goal is functional (gene/pathway) profiling rather than taxonomic profiling
  — bioBakery tools such as HUMAnN are the appropriate counterpart
- The community is dominated by viral or eukaryotic microbes, which the current marker
  approach does not extensively incorporate due to their unique genomic architectures
  and quality-control requirements
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: MetaPhlAn 4
  role: Primary clade-specific marker-gene profiler producing species-level (SGB-level)
    relative abundances and unclassified-read estimates from shotgun metagenomes
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn via bioconda
- name: bioBakery
  role: Integrated suite/wiki providing MetaPhlAn tutorials and documentation used
    in the profiling workflow
  repo: https://github.com/biobakery/biobakery
- name: PhyloPhlAn
  role: Provides workflows for phylogenetic characterization of profiled SGBs/uSGBs
- name: StrainPhlAn
  role: Strain-level microbial population genomics available within MetaPhlAn, enabling
    single-strain-resolution profiling of uncharacterized taxa
  repo: https://github.com/biobakery/MetaPhlAn
merged_aliases:
- metagenomic-taxonomic-profile-benchmark-comparison
merged_alias_records:
- alias: metagenomic-taxonomic-profile-benchmark-comparison
  slug: metagenomic-taxonomic-profile-benchmark-comparison
  cosine_score: 0.8074
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_003
  - task_004
  - task_006
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:be736c663b283a9c67e4162a069777d8dd9d9bd6b6d6e746889649063c7bc7e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - PhyloPhlAn
  - StrainPhlAn
  - bioBakery
  - conda
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# metagenomic-taxonomic-profiling

## Summary

Species-level taxonomic profiling of metagenomic shotgun sequencing data using MetaPhlAn 4's clade-specific marker genes, which cover 26,970 species-level genome bins (SGBs) including 4,992 taxonomically unidentified uSGBs, enabling accurate detection and quantification of organisms without cultured isolates.

## When to use

Use when shotgun metagenomic FASTQ samples need species-/SGB-level relative abundance profiles, especially when uncharacterized species (uSGBs), read-explainability gains, or prevalence estimates across large sample collections are the analysis goal.

## When NOT to use

- Input is already a taxonomic abundance/feature table — no re-profiling with marker genes is needed
- The goal is functional (gene/pathway) profiling rather than taxonomic profiling — bioBakery tools such as HUMAnN are the appropriate counterpart
- The community is dominated by viral or eukaryotic microbes, which the current marker approach does not extensively incorporate due to their unique genomic architectures and quality-control requirements

## Inputs

- Shotgun metagenomic sequencing reads (FASTQ) from human gut, airways, wild mice, rumen, ocean, and other environments
- Synthetic metagenomes with ground-truth per-SGB abundance compositions (65 kSGB/uSGB synthetic + 5 mixed new-MAG samples)
- MetaPhlAn 4 marker database of 26,970 SGBs (4,992 uSGBs)
- Sample metadata (environment, lifestyle westernized/nonwesternized, age childhood/adult) for stratified analyses

## Outputs

- Per-sample species-level (SGB-level) relative abundance profiles from MetaPhlAn 4
- Per-SGB F1 scores combining detection and quantification performance (f1_scores.csv)
- Per-environment fold increases in explainable reads and uSGB richness fractions (environment_read_explainability_summary.tsv)
- Stratified SGB prevalence table by lifestyle and age (sgb_prevalence_by_lifestyle_age.csv)

## How to apply

Install MetaPhlAn via conda (`conda install -c bioconda metaphlan`) and run it on each FASTQ to produce species-level (SGB-level) relative abundance profiles using the MetaPhlAn 4 marker database built from 1.01M prokaryotic reference and metagenome-assembled genomes, with the unclassified-reads estimation feature enabled to compute explainable-read fractions. To validate quantification of uncharacterized species, profile synthetic metagenomes with known per-SGB ground-truth compositions and match predicted SGBs against expected compositions, computing per-SGB F1 scores that combine detection (presence/absence) and quantification (estimated vs expected abundance) — expected anchors are uSGB 0.97±0.02, kSGB 0.96±0.024, and mixed-evaluation MAGs 0.98±0.006. For large collections (e.g. the 24,515-sample, 145-dataset set), aggregate profiles by environment and compute fold increases in explainable reads versus a baseline profiler (MetaPhlAn 3.0.12), expecting ~20% in human gut, 1.95-fold airways, 3.26-fold wild mice, 14.15-fold rumen, 6.65-fold ocean. Prevalence analyses rank SGBs by detection fraction across lifestyle/age strata, with top westernized-gut taxa Blautia wexlerae (SGB4837, 89.2%), the Bacteroides uniformis group (SGB1836, 88.1%), and Phocaeicola vulgatus (SGB1814, 85.8%). Write stratified results to structured tables (e.g. f1_scores.csv, environment_read_explainability_summary.tsv, sgb_prevalence_by_lifestyle_age.csv) and verify each against the reported anchor values.

## Related tools

- **MetaPhlAn 4** (Primary clade-specific marker-gene profiler producing species-level (SGB-level) relative abundances and unclassified-read estimates from shotgun metagenomes) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn via bioconda)
- **bioBakery** (Integrated suite/wiki providing MetaPhlAn tutorials and documentation used in the profiling workflow) — https://github.com/biobakery/biobakery
- **PhyloPhlAn** (Provides workflows for phylogenetic characterization of profiled SGBs/uSGBs)
- **StrainPhlAn** (Strain-level microbial population genomics available within MetaPhlAn, enabling single-strain-resolution profiling of uncharacterized taxa) — https://github.com/biobakery/MetaPhlAn

## Examples

```
conda install -c bioconda metaphlan; metaphlan sample.fastq --input_type fastq --unclassified_estimation -o sample_profile.tsv
```

## Evaluation signals

- Per-SGB F1 scores on synthetic metagenomes fall within reported ranges: uSGB 0.97±0.02, kSGB 0.96±0.024, mixed-evaluation MAGs 0.98±0.006
- f1_scores.csv contains one F1 row per evaluated SGB
- Read-explainability fold increases reproduce reported anchors (rumen 14.15-fold, ocean 6.65-fold, wild mice 3.26-fold, airways 1.95-fold, ~20% human gut)
- Per-sample uSGB richness fraction matches mean 23.13% (s.d. 17.89%)
- Top-ranked gut SGB prevalences match Blautia wexlerae SGB4837 89.2%, Bacteroides uniformis group SGB1836 88.1%, Phocaeicola vulgatus SGB1814 85.8%, with uSGB prevalence higher in nonwesternized and childhood samples

## Limitations

- Even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats; environmental, nonhost-associated, and under-studied communities remain enriched for sequences not captured by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences due to their unique genomic architectures and quality-control requirements
- A tradeoff between sensitivity and specificity when applying quality-control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge
- Further integration of isolate genomes and new methods for defining taxonomic clades above the microbial family level are still needed

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [abstract] Read-explainability gains in less-characterized environments: "MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome"
- [readme] Conda installation of MetaPhlAn: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [intro] Scope of the profiler: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] Remaining gaps in under-characterized habitats: "Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats."
