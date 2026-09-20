---
name: cross-platform-16s-vs-mwgs-concordance-analysis
description: Use when comparing 16S-derived genus/species abundance profiles (full-length
  V1–V9 or partial V1–V3 amplicons) against matched mWGS species abundances for the
  same samples, to judge whether a 16S region/platform combination achieves species-level
  resolution.
when_to_use_negative:
- The goal is strain-level discrimination via phasing of 16S gene SNPs/substitution
  profiles rather than OTU-based species abundance estimation
- Only one sequencing platform is available and no matched mWGS data exist, so no
  cross-platform concordance can be assessed
- Input data are already clustered OTU tables or abundance profiles rather than raw
  preprocessed amplicon read sets
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: USEARCH
  role: Dereplication and 99%-identity OTU clustering of amplicon reads, plus alignment
    of Bacteroides OTU representative sequences to the RTG reference 16S database
    for species-level taxonomy assignment
- name: PacBio
  role: Long-read platform providing full-length (V1–V9) 16S amplicon sequencing whose
    99% OTUs yield species-level Bacteroides abundances concordant with mWGS
- name: ruffus (CGAT framework)
  role: Pipeline framework organizing the repository's per-figure analysis code underlying
    the Fig. 1–3 comparisons
  repo: https://github.com/cgat-developers/ruffus
- name: weinstock_full_length_16s
  role: Source repository of preprocessed V1–V9 and V1–V3 stool amplicon read sets
    and analysis pipelines
  repo: https://github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:e248db02730508ae77cedea3aaf755e1b155c7a6b7fc6dc1823110f394b8d446
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - USEARCH
  - ruffus (CGAT framework)
  - weinstock_full_length_16s
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# cross-platform-16s-vs-mwgs-concordance-analysis

## Summary

Quantifies Bacteroides genus- and species-level abundances from 99%-identity OTU clustering of full-length (V1–V9, PacBio) and partial (V1–V3, MiSeq) 16S amplicons and assesses their concordance with matched metagenomic whole-genome-shotgun (mWGS) quantification in the same stool samples.

## When to use

Use when comparing 16S-derived genus/species abundance profiles (full-length V1–V9 or partial V1–V3 amplicons) against matched mWGS species abundances for the same samples, to judge whether a 16S region/platform combination achieves species-level resolution.

## When NOT to use

- The goal is strain-level discrimination via phasing of 16S gene SNPs/substitution profiles rather than OTU-based species abundance estimation
- Only one sequencing platform is available and no matched mWGS data exist, so no cross-platform concordance can be assessed
- Input data are already clustered OTU tables or abundance profiles rather than raw preprocessed amplicon read sets

## Inputs

- Preprocessed PacBio full-length (V1–V9) 16S amplicon reads from human stool samples (deposited at github.com/TheJacksonLaboratory/weinstock_full_length_16s)
- Preprocessed MiSeq partial (V1–V3) 16S amplicon reads from the same samples
- RTG reference 16S database for species-level taxonomy assignment
- Matched metagenomic whole-genome-shotgun (mWGS) species-level abundance profiles for the same individuals

## Outputs

- 99%-identity OTU tables per sequencing platform (V1–V9 and V1–V3)
- Bacteroides OTU representative sequences with species-level taxonomic assignments
- Genus- and species-level Bacteroides relative abundance profiles per individual for each 16S region/platform
- Cross-platform concordance comparison (Fig. 3a–b style) of 16S vs mWGS abundances per individual

## How to apply

Load the preprocessed stool amplicon read sets (PacBio V1–V9 full-length and MiSeq V1–V3) from the deposited repository github.com/TheJacksonLaboratory/weinstock_full_length_16s, dereplicate, and cluster each platform's reads into OTUs at 99% sequence identity with USEARCH. Align Bacteroides OTU representative sequences to the RTG reference 16S database with USEARCH to assign species-level taxonomy per OTU, then compute genus- and species-level relative abundance profiles from the OTU tables. Compare the 16S-derived abundances with matched mWGS quantification per individual, reproducing the paper's Fig. 3a–b comparisons. The rationale is that full-length 16S sequences clustered at 99% identity were shown to provide reasonable estimates of Bacteroides species relative abundance, whereas partial V1–V3 reads may miss species entirely. Concordance is judged at the individual level (high- vs low-Bacteroides grouping) and at the species level (presence/absence of abundant taxa such as Bacteroides intestinalis).

## Related tools

- **USEARCH** (Dereplication and 99%-identity OTU clustering of amplicon reads, plus alignment of Bacteroides OTU representative sequences to the RTG reference 16S database for species-level taxonomy assignment)
- **PacBio** (Long-read platform providing full-length (V1–V9) 16S amplicon sequencing whose 99% OTUs yield species-level Bacteroides abundances concordant with mWGS)
- **ruffus (CGAT framework)** (Pipeline framework organizing the repository's per-figure analysis code underlying the Fig. 1–3 comparisons) — https://github.com/cgat-developers/ruffus
- **weinstock_full_length_16s** (Source repository of preprocessed V1–V9 and V1–V3 stool amplicon read sets and analysis pipelines) — https://github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Evaluation signals

- Full-length (V1–V9) 99% OTU-based Bacteroides species relative abundances provide reasonable estimates of the mWGS-measured species abundances, per the paper's reported finding
- Individuals group concordantly as high- vs low-Bacteroides between the 16S-derived profiles and mWGS quantification
- Bacteroides intestinalis is absent from the V1–V3 MiSeq profiles in the sample where mWGS and full-length 16S detect it as abundant (expected negative control for the partial region)
- Each OTU representative sequence receives a species-level assignment from the RTG reference 16S database alignment before abundance aggregation
- Both V1–V9 and V1–V3 OTU tables reflect the species-level Bacteroides variation observed by mWGS, matching the Fig. 3b pattern

## Limitations

- V1–V3 partial-amplicon sequencing failed to detect the abundant Bacteroides intestinalis in one stool sample, so partial-region concordance is incomplete at species level
- Targeting 16S variable regions with short-read platforms cannot achieve the taxonomic resolution of sequencing the entire ~1500 bp gene
- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences, which motivated ignoring insertions/deletions and focusing on substitutions
- The study stopped short of determining how well the 99% identity threshold separates intragenomic vs inter-genomic variation for all bacterial species, so the threshold is validated primarily for Bacteroides

## Evidence

- [discussion] full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [abstract] We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene.: "We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene."
- [readme] Code is organised into a set of ruffus pipelines pertaining to different figures.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
- [abstract] appropriate treatment of full-length 16S intragenomic copy variants has the potential to provide taxonomic resolution of bacterial communities at species and strain level.: "appropriate treatment of full-length 16S intragenomic copy variants has the potential to provide taxonomic resolution of bacterial communities at species and strain level."
