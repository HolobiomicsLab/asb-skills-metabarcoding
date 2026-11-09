---
name: sgb-prevalence-estimation
description: Use when a cohort of shotgun metagenomes (e.g. ~19.5k curatedMetagenomicData
  gut samples) must be profiled to rank SGBs by detection frequency across lifestyle
  (westernized vs nonwesternized) or age (childhood vs adult) strata.
when_to_use_negative:
- Input is already a species/SGB abundance or prevalence table and only stratification
  or ranking is needed — re-profiling with MetaPhlAn 4 is unnecessary
- The goal is functional profiling (genes/pathways) rather than taxonomic SGB prevalence
  — use HUMAnN-style workflows instead
- Samples are from environmental, nonhost-associated habitats where sequences are
  highly enriched for those not captured by current uSGBs, making SGB prevalence estimates
  unreliable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: MetaPhlAn 4
  role: Profiles each shotgun metagenome to SGB-level relative abundances using its
    unique marker gene database, enabling prevalence computation per SGB
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn (bioconda) used to set up the profiling
    environment
- name: bioBakery
  role: Integrated platform and tutorials providing MetaPhlAn and dependencies for
    the profiling workflow
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:b381fbbcf429d88eece1f1978892f5763dbcd708b7c44315d623d38cdbedfcb5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - bioBakery
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# sgb-prevalence-estimation

## Summary

Estimate the prevalence of species-level genome bins (SGBs) across large human gut metagenome collections by profiling samples with MetaPhlAn 4 (26,970 SGBs from ~1.01 M reference and metagenome-assembled genomes) and computing the fraction of samples in which each SGB is detected, stratified by lifestyle and age categories.

## When to use

Use when a cohort of shotgun metagenomes (e.g. ~19.5k curatedMetagenomicData gut samples) must be profiled to rank SGBs by detection frequency across lifestyle (westernized vs nonwesternized) or age (childhood vs adult) strata.

## When NOT to use

- Input is already a species/SGB abundance or prevalence table and only stratification or ranking is needed — re-profiling with MetaPhlAn 4 is unnecessary
- The goal is functional profiling (genes/pathways) rather than taxonomic SGB prevalence — use HUMAnN-style workflows instead
- Samples are from environmental, nonhost-associated habitats where sequences are highly enriched for those not captured by current uSGBs, making SGB prevalence estimates unreliable

## Inputs

- Human gut shotgun metagenome samples from the curatedMetagenomicData gut subset (19,468 samples)
- Metadata assigning each sample to a lifestyle category (westernized vs nonwesternized) and an age category (childhood vs adult)
- MetaPhlAn 4 marker database covering 26,970 SGBs

## Outputs

- Per-sample MetaPhlAn 4 SGB-level relative abundance profiles
- Stratified SGB prevalence table (e.g. sgb_prevalence_by_lifestyle_age.csv) with per-stratum detection fractions
- Ranked list of top-prevalence SGBs and uSGB enrichment comparisons between strata

## How to apply

Retrieve the curatedMetagenomicData gut subset (19,468 human gut shotgun metagenomes) as the profiling input cohort. Run MetaPhlAn 4 on each sample using its marker database (unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically unidentified uSGBs) to obtain species-level/SGB-level relative abundance profiles. Aggregate per-sample SGB profiles and compute prevalence as the fraction of samples in which each SGB is detected, separately within each lifestyle category (westernized vs nonwesternized) and each age category (childhood vs adult). Compare uSGB prevalence between nonwesternized and westernized populations and between childhood and adult samples to quantify enrichment, then rank SGBs by overall prevalence and write the stratified prevalence table (e.g. sgb_prevalence_by_lifestyle_age.csv). Install MetaPhlAn via conda (`conda install -c bioconda metaphlan`) as part of the bioBakery platform.

## Related tools

- **MetaPhlAn 4** (Profiles each shotgun metagenome to SGB-level relative abundances using its unique marker gene database, enabling prevalence computation per SGB) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn (bioconda) used to set up the profiling environment)
- **bioBakery** (Integrated platform and tutorials providing MetaPhlAn and dependencies for the profiling workflow) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan && metaphlan sample_S1.fastq.bz2 --input_type fastq --bowtie2db metaphlan_databases/mpa_vOct22_CHOCOPhlAnSGB_202212 -o profile_S1.txt
```

## Evaluation signals

- Top-ranked taxa and prevalences match reported values: Blautia wexlerae (SGB4837, 89.2%), Bacteroides uniformis group (SGB1836, 88.1%), Phocaeicola vulgatus (SGB1814, 85.8%)
- uSGB (taxonomically unidentified SGB) prevalence is higher in nonwesternized populations than in westernized populations
- uSGB prevalence is higher in childhood samples than in adult samples
- Per-sample profiles contain a single relative abundance value per SGB and sums are consistent with MetaPhlAn 4 relative abundance output
- Prevalence table covers all strata and detection fractions fall in [0, 1]

## Limitations

- 4,992 of the 26,970 SGBs are taxonomically unidentified at the species level, so many prevalent bins can only be reported as uSGBs
- Under-characterized habitats remain poorly profiled even with the extended MetaPhlAn 4 SGB and marker set
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences, so SGB prevalence covers mainly bacterial/archaeal taxa
- A tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge

## Evidence

- [abstract] From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] Application to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [readme] To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [discussion] Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats.: "Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats."
- [intro] MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
