---
name: age-category-stratified-taxa-prevalence
description: Use when you have per-sample species/SGB relative-abundance profiles
  from a large metagenome cohort annotated with host metadata (lifestyle, age) and
  need prevalence-ranked taxa or enrichment of unidentified taxa (uSGBs) per category.
when_to_use_negative:
- Input is already an aggregated feature/abundance table without raw reads — MetaPhlAn
  4 profiling from shotgun reads is the prerequisite step
- The goal is strain-level genomic or population-genetic analysis (use StrainPhlAn)
  rather than taxon prevalence across sample strata
- Samples come from environmental, nonhost-associated, or under-characterized habitats,
  where sequences are highly enriched for content not captured by current uSGBs
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaPhlAn 4
  role: Produces per-sample species/SGB-level relative abundance profiles that are
    aggregated into prevalence estimates per lifestyle and age category
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn via bioconda before running the profiling/prevalence
    workflow
- name: bioBakery
  role: Integrated meta'omic profiling environment/tutorials providing the MetaPhlAn
    tooling used for taxonomic profiling
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:c1ce1171b94d4fc8fffff23193491143382a046e00d90eae2fc6768174379b31
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - bioBakery
  - conda
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# age-category-stratified-taxa-prevalence

## Summary

Computes and stratifies taxon (SGB-level) prevalence — the fraction of samples in which each taxon is detected — across host categories such as lifestyle (westernized vs nonwesternized) and age (childhood vs adult), using MetaPhlAn 4 profiles of large human gut metagenome collections.

## When to use

Use when you have per-sample species/SGB relative-abundance profiles from a large metagenome cohort annotated with host metadata (lifestyle, age) and need prevalence-ranked taxa or enrichment of unidentified taxa (uSGBs) per category.

## When NOT to use

- Input is already an aggregated feature/abundance table without raw reads — MetaPhlAn 4 profiling from shotgun reads is the prerequisite step
- The goal is strain-level genomic or population-genetic analysis (use StrainPhlAn) rather than taxon prevalence across sample strata
- Samples come from environmental, nonhost-associated, or under-characterized habitats, where sequences are highly enriched for content not captured by current uSGBs

## Inputs

- Human gut shotgun metagenomes (FASTQ), e.g. the 19,468-sample curatedMetagenomicData gut subset
- Per-sample metadata with lifestyle category (westernized/nonwesternized) and age category (childhood/adult)
- MetaPhlAn 4 marker database (26,970 SGBs, 4,992 taxonomically unidentified)

## Outputs

- Stratified SGB prevalence table (CSV, e.g. sgb_prevalence_by_lifestyle_age.csv) with prevalence per lifestyle and age category
- Ranked list of top-prevalence taxa with prevalences (e.g. Blautia wexlerae SGB4837 89.2%)
- uSGB prevalence comparison (nonwesternized vs westernized; childhood vs adult)

## How to apply

Profile each shotgun metagenome with MetaPhlAn 4, whose marker set covers 26,970 species-level genome bins (4,992 taxonomically unidentified uSGBs), to obtain per-sample SGB relative abundances. Aggregate profiles and define prevalence per SGB as the fraction of samples in which it is detected, computed within each lifestyle category (westernized vs nonwesternized) and each age category (childhood vs adult). Rank SGBs by overall prevalence and record the top taxa; additionally compare uSGB prevalence between nonwesternized vs westernized and childhood vs adult strata to quantify enrichment of previously uncharacterized organisms. Expected reference results for the 19,468-sample curatedMetagenomicData gut subset: Blautia wexlerae (SGB4837) 89.2%, the Bacteroides uniformis group (SGB1836) 88.1%, and Phocaeicola vulgatus (SGB1814) 85.8% as the most prevalent taxa in westernized populations. Write the stratified prevalence table (e.g. sgb_prevalence_by_lifestyle_age.csv) and validate against these reference values and the uSGB enrichment direction.

## Related tools

- **MetaPhlAn 4** (Produces per-sample species/SGB-level relative abundance profiles that are aggregated into prevalence estimates per lifestyle and age category) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn via bioconda before running the profiling/prevalence workflow)
- **bioBakery** (Integrated meta'omic profiling environment/tutorials providing the MetaPhlAn tooling used for taxonomic profiling) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan; metaphlan sample.fastq.bz2 --input_type fastq --bowtie2db metaphlan_databases/mpa_vJan21_CHOCOPhlAnSGB_202103 -o sample_sgb_profile.tsv
```

## Evaluation signals

- Top-ranked taxa match reported values: Blautia wexlerae SGB4837 89.2%, Bacteroides uniformis group SGB1836 88.1%, Phocaeicola vulgatus SGB1814 85.8%
- Prevalence is a fraction bounded in [0,1] within each stratum and computed on the full cohort (19,468 gut metagenomes)
- uSGB prevalence is higher in nonwesternized populations than in westernized populations
- uSGB prevalence is higher in childhood samples than in adult samples
- Output CSV contains one prevalence value per SGB per lifestyle and age category with consistent SGB identifiers

## Limitations

- 4,992 of the 26,970 SGBs remain taxonomically unidentified (uSGBs), so prevalence of many prevalent organisms cannot be tied to species names
- Under-studied habitats remain highly enriched for sequences not captured by current uSGBs, biasing prevalence estimates for non-gut or environmental samples
- Prevalence stratification depends on reliable host metadata labeling of lifestyle and age categories; mislabeled samples distort the stratum-level estimates

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, 4,992 of which are taxonomically unidentified at the species level.: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] Application to >24,500 metagenomes shows previously undetected species are strong biomarkers for host conditions and lifestyles.: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes and shows that even"
- [intro] MetaPhlAn 4 is a computational tool for species-level microbial profiling from metagenomic shotgun sequencing data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] Environmental and under-studied communities are enriched for sequences not captured by current uSGBs, limiting prevalence estimates there.: "Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs"
- [readme] MetaPhlAn can be installed through conda from bioconda.: "To install MetaPhlan through conda run:

``n$ conda install -c bioconda metaphlan
"
