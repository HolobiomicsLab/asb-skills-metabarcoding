---
name: unidentified-species-bin-quantification
description: Use when profiling shotgun metagenomes with MetaPhlAn 4 and the analysis
  goal is to quantify or compare prevalence/abundance of uncharacterized (unidentified)
  SGBs (uSGBs) across cohorts, lifestyles, or host conditions.
when_to_use_negative:
- When the input is already a species/feature abundance table rather than raw shotgun
  reads — MetaPhlAn 4 profiling of FASTQ input is the prerequisite step, not re-applicable.
- When the target community is environmental, nonhost-associated, or under-studied,
  where sequences may not be captured even by current uSGBs per the article's discussion.
- When the question is functional profiling (genes/pathways) rather than taxonomic
  SGB prevalence — tools like HUMAnN, not SGB prevalence quantification, apply.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0637
tools:
- name: MetaPhlAn 4
  role: 'Primary profiler: computes SGB-level relative abundances from shotgun reads
    using unique marker genes for 26,970 SGBs, including 4,992 taxonomically unidentified
    uSGBs'
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn via bioconda
- name: bioBakery
  role: Integrated platform/virtual image in which MetaPhlAn 4 and related meta'omic
    profiling tools are delivered
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:b2e003292bea2b66d19ddfcdd51a5c9b0d60c397830369f22c6061c219e6746c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - bioBakery
  - conda
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# unidentified-species-bin-quantification

## Summary

Quantify the prevalence and abundance of taxonomically unidentified species-level genome bins (uSGBs) in metagenomic cohorts profiled with MetaPhlAn 4, whose extended marker set covers 26,970 SGBs from 1.01M reference and metagenome-assembled genomes, 4,992 of which are taxonomically unidentified at the species level.

## When to use

Use when profiling shotgun metagenomes with MetaPhlAn 4 and the analysis goal is to quantify or compare prevalence/abundance of uncharacterized (unidentified) SGBs (uSGBs) across cohorts, lifestyles, or host conditions.

## When NOT to use

- When the input is already a species/feature abundance table rather than raw shotgun reads — MetaPhlAn 4 profiling of FASTQ input is the prerequisite step, not re-applicable.
- When the target community is environmental, nonhost-associated, or under-studied, where sequences may not be captured even by current uSGBs per the article's discussion.
- When the question is functional profiling (genes/pathways) rather than taxonomic SGB prevalence — tools like HUMAnN, not SGB prevalence quantification, apply.

## Inputs

- Human gut shotgun metagenome FASTQ files (e.g. the 19,468-sample curatedMetagenomicData gut subset)
- MetaPhlAn 4 marker gene database (26,970 SGBs, 4,992 taxonomically unidentified)
- Per-sample metadata with lifestyle and age strata (westernized/nonwesternized, childhood/adult)

## Outputs

- Per-sample SGB-level relative abundance profiles (MetaPhlAn 4 output)
- Stratified SGB prevalence table (e.g. sgb_prevalence_by_lifestyle_age.csv) with prevalence per lifestyle and age category
- Ranked list of top-prevalence SGBs and uSGB-vs-known-SGB prevalence comparisons across strata

## How to apply

Run MetaPhlAn 4 with its marker database (built from 1.01M curated prokaryotic reference and metagenome-assembled genomes defining unique markers for 26,970 species-level genome bins) on each shotgun metagenome to obtain species-/SGB-level relative abundance profiles. Aggregate per-sample profiles and compute prevalence as the fraction of samples in which each SGB is detected within each stratum (e.g. westernized vs nonwesternized, childhood vs adult). SGBs that are taxonomically unidentified at the species level are the uSGBs (4,992 in the MetaPhlAn 4 database); track their prevalence separately from known-species SGBs. Compare uSGB prevalence across strata to quantify enrichment of uncharacterized taxa, and rank all SGBs by overall prevalence to surface the top-known taxa (e.g. Blautia wexlerae SGB4837 89.2%, Bacteroides uniformis group SGB1836 88.1%, Phocaeicola vulgatus SGB1814 85.8%). Rationale: MetaPhlAn 4's uSGB markers extend profiling to organisms with no cultured isolates, so prevalence of uSGBs is a direct measure of how much uncharacterized diversity the cohort harbors.

## Related tools

- **MetaPhlAn 4** (Primary profiler: computes SGB-level relative abundances from shotgun reads using unique marker genes for 26,970 SGBs, including 4,992 taxonomically unidentified uSGBs) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn via bioconda)
- **bioBakery** (Integrated platform/virtual image in which MetaPhlAn 4 and related meta'omic profiling tools are delivered) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan && metaphlan sample.fastq --input_type fastq --bowtie2db metaphlan_databases/mpa_vJan21_CHOCOPhlAnSGB_202103 --nproc 8 > sample_sgb_profile.txt
```

## Evaluation signals

- Top-ranked known SGBs and prevalences match reported values: Blautia wexlerae SGB4837 89.2%, Bacteroides uniformis group SGB1836 88.1%, Phocaeicola vulgatus SGB1814 85.8%
- uSGB prevalence is higher in nonwesternized populations than in westernized populations
- uSGB prevalence is higher in childhood samples than in adult samples
- Prevalence values are bounded in [0,1] and the stratified table (sgb_prevalence_by_lifestyle_age.csv) covers all samples and strata of the input cohort (e.g. 19,468 gut metagenomes)
- MetaPhlAn 4 database version reports 26,970 SGBs with 4,992 taxonomically unidentified bins, so uSGB labels in output are consistent with the database version used

## Limitations

- Environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs.
- A tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge.
- Further integration of isolate genomes and new methods for defining taxonomic clades above the microbial family level are still needed; new databases (~2 per year planned) will expand profilable diversity.

## Evidence

- [abstract] From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] Application to >24,500 metagenomes shows uncharacterized species as lifestyle biomarkers: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes and shows that even"
- [intro] MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs: "Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs"
- [readme] To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan: "To install MetaPhlan through conda run:

``n$ conda install -c bioconda metaphlan"
