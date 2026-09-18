---
name: lifestyle-stratified-microbiome-analysis
description: Use when a large collection of human gut metagenomes needs species/SGB-level
  profiling with MetaPhlAn 4 and prevalence ranking stratified by host lifestyle or
  age category, e.g. reproducing prevalence panels across westernized vs nonwesternized
  populations.
when_to_use_negative:
- The input is already a species/SGB abundance or prevalence feature table and no
  re-profiling of reads is needed.
- The samples are environmental, nonhost-associated, or under-studied microbial communities,
  which the paper notes remain highly enriched for sequences not captured even by
  current uSGBs.
- The goal is functional profiling (genes/pathways) rather than taxonomic SGB prevalence
  stratification.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaPhlAn 4
  role: Runs species-level/SGB-level taxonomic profiling on each shotgun metagenome
    to produce the relative abundance profiles used for prevalence computation
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn via bioconda
- name: bioBakery
  role: Integrated platform/virtual image providing MetaPhlAn and supporting tools
    for the meta'omic profiling workflow
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:0257a8577024bee16f4f73e10303b3cc2f1b04731204d98a288bac00eb7d8f9c
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
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# lifestyle-stratified-microbiome-analysis

## Summary

Profiles large cohorts of human gut shotgun metagenomes with MetaPhlAn 4 to quantify species-level genome bin (SGB) prevalence, then stratifies prevalence by lifestyle (westernized vs nonwesternized) and age (childhood vs adult) to identify the most prevalent taxa and lifestyle/age-enriched uncharacterized species (uSGBs).

## When to use

Use when a large collection of human gut metagenomes needs species/SGB-level profiling with MetaPhlAn 4 and prevalence ranking stratified by host lifestyle or age category, e.g. reproducing prevalence panels across westernized vs nonwesternized populations.

## When NOT to use

- The input is already a species/SGB abundance or prevalence feature table and no re-profiling of reads is needed.
- The samples are environmental, nonhost-associated, or under-studied microbial communities, which the paper notes remain highly enriched for sequences not captured even by current uSGBs.
- The goal is functional profiling (genes/pathways) rather than taxonomic SGB prevalence stratification.

## Inputs

- Human gut shotgun metagenome sequencing samples (e.g. 19,468 samples from the curatedMetagenomicData gut subset)
- Sample metadata with lifestyle category (westernized/nonwesternized) and age category (childhood/adult)
- MetaPhlAn 4 marker database (26,970 species-level genome bins, 4,992 taxonomically unidentified)

## Outputs

- Per-sample SGB/species-level relative abundance profiles from MetaPhlAn 4
- Stratified prevalence table (e.g. sgb_prevalence_by_lifestyle_age.csv) with per-category prevalence fractions
- Ranked list of most prevalent SGBs and uSGB enrichment comparisons across lifestyle and age strata

## How to apply

Retrieve the gut metagenome cohort (e.g. the 19,468 human gut shotgun metagenomes from the curatedMetagenomicData gut subset) and run MetaPhlAn 4 on each sample with the MetaPhlAn 4 marker database, which defines unique markers for 26,970 SGBs (4,992 taxonomically unidentified) from 1.01 M reference and metagenome-assembled genomes. Aggregate the per-sample SGB relative-abundance profiles and compute prevalence as the fraction of samples in which each SGB is detected, separately within each lifestyle category (westernized vs nonwesternized) and each age category (childhood vs adult). Compare uSGB (taxonomically unidentified SGB) prevalence between nonwesternized and westernized samples and between childhood and adult samples to quantify enrichment of previously undetected species. Rank SGBs by overall prevalence and record top-ranked taxa; write the stratified table (e.g. sgb_prevalence_by_lifestyle_age.csv). Validate against reported top taxa and their prevalences.

## Related tools

- **MetaPhlAn 4** (Runs species-level/SGB-level taxonomic profiling on each shotgun metagenome to produce the relative abundance profiles used for prevalence computation) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn via bioconda)
- **bioBakery** (Integrated platform/virtual image providing MetaPhlAn and supporting tools for the meta'omic profiling workflow) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan && metaphlan sample.fastq.gz --input_type fastq --bowtie2db metaphlan_databases/mpa_vJan21_CHOCOPhlAnSGB_202103 --nproc 8 > sample_sgb_profile.tsv
```

## Evaluation signals

- Top-ranked SGBs and their computed prevalences match reported values: Blautia wexlerae (SGB4837, 89.2%), Bacteroides uniformis group (SGB1836, 88.1%), Phocaeicola vulgatus (SGB1814, 85.8%).
- uSGB prevalence is higher in nonwesternized populations than in westernized populations.
- uSGB prevalence is higher in childhood samples than in adult samples.
- All samples in the cohort yield a MetaPhlAn 4 profile and per-category prevalence values fall within [0, 1].
- The stratified prevalence table (sgb_prevalence_by_lifestyle_age.csv) covers every lifestyle and age category defined in the metadata.

## Limitations

- Environmental, nonhost-associated, and under-studied microbial communities remain highly enriched for sequences not captured even by current uSGBs.
- Further work remains to better profile under-characterized habitats even with the extended MetaPhlAn 4 SGB and marker set.
- The current methods do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements.
- A tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge.

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, 4,992 of which are taxonomically unidentified at the species level.: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] Previously undetected species are strong biomarkers for host conditions and lifestyles across tens of thousands of metagenomes.: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes and shows that even"
- [results] The profiled cohort consists of curated human gut shotgun metagenomes quantified with MetaPhlAn 4.: [evidence span withheld — anchor status: not_found]
- [results] Reported top prevalent SGBs with their prevalence percentages.: [evidence span withheld — anchor status: not_found]
- [readme] MetaPhlAn 4 installation via conda as documented in the README.: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [discussion] Limitation on viral and eukaryotic sequences not extensively incorporated.: "The current methods also do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements relative to bacterial and"
