---
name: usearch-reference-alignment-species-assignment
description: Use when OTU representative sequences (clustered at 99% identity from
  full-length or partial 16S amplicons) need species-level taxonomic labels to compare
  16S-derived genus/species abundances against metagenomic (mWGS) quantification.
when_to_use_negative:
- Reads have not been clustered into OTUs at 99% identity — align raw amplicon reads
  directly will conflate intragenomic copy variants with true species differences
- The target taxa's species discrimination relies on insertion/deletion variation
  between intragenomic 16S copies — the approach focuses on substitutions and ignores
  indels
- Input is already a species-assigned feature/biom table — no reference alignment
  is needed
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: USEARCH
  role: Dereplication, 99%-identity OTU clustering of amplicon reads, and alignment
    of Bacteroides OTU representative sequences to the RTG reference 16S database
    for species assignment
- name: PacBio
  role: Long-read platform generating the full-length V1–V9 16S amplicon reads that
    feed the OTU clustering and reference-alignment species assignment
- name: ruffus (CGAT framework)
  role: Python pipeline framework organising the figure-specific analysis code, including
    the pipelines underlying Fig. 3 comparisons
  repo: https://github.com/cgat-developers/ruffus
- name: weinstock_full_length_16s
  role: Depository of the analysis code and preprocessed stool amplicon read sets
    used in the workflow
  repo: https://github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:5b417154b23820c6e978e67c6aa00be7d15d7541c367afe31663732ee2f20630
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - USEARCH
  - ruffus (CGAT framework)
  - weinstock_full_length_16s
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# usearch-reference-alignment-species-assignment

## Summary

Assigns species-level taxonomy to 16S OTU representative sequences by aligning them against a curated full-length 16S reference database (RTG) with USEARCH, enabling species-level Bacteroides quantification from 99%-identity OTU tables of full-length (V1–V9, PacBio) and partial (V1–V3, MiSeq) amplicons.

## When to use

Use when OTU representative sequences (clustered at 99% identity from full-length or partial 16S amplicons) need species-level taxonomic labels to compare 16S-derived genus/species abundances against metagenomic (mWGS) quantification.

## When NOT to use

- Reads have not been clustered into OTUs at 99% identity — align raw amplicon reads directly will conflate intragenomic copy variants with true species differences
- The target taxa's species discrimination relies on insertion/deletion variation between intragenomic 16S copies — the approach focuses on substitutions and ignores indels
- Input is already a species-assigned feature/biom table — no reference alignment is needed

## Inputs

- Dereplicated 16S amplicon read sets (PacBio V1–V9 full-length; MiSeq V1–V3) from stool samples, e.g. from github.com/TheJacksonLaboratory/weinstock_full_length_16s
- 99%-identity OTU representative sequences (FASTA)
- RTG reference 16S database with species-level taxonomy

## Outputs

- Species-level taxonomic assignments for each Bacteroides OTU representative sequence
- Genus- and species-level Bacteroides relative abundance profiles per sample per platform/region
- Comparison against matched mWGS species abundances (Fig. 3a-b style)

## How to apply

First dereplicate and cluster each platform's preprocessed amplicon reads (PacBio V1–V9 full-length; MiSeq V1–V3) into OTUs at 99% sequence identity using USEARCH. Then take the Bacteroides OTU representative sequences and align them to the RTG reference 16S database with USEARCH to assign species-level taxonomy per OTU. Aggregate OTU counts into genus- and species-level relative abundance profiles per sample. The rationale, per Johnson et al. 2019, is that full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance, so species assignment quality depends on both the 99% clustering threshold and accurate reference alignment. Compare the resulting 16S-derived profiles with matched mWGS species quantification (Fig. 3a-b) to validate concordance.

## Related tools

- **USEARCH** (Dereplication, 99%-identity OTU clustering of amplicon reads, and alignment of Bacteroides OTU representative sequences to the RTG reference 16S database for species assignment)
- **PacBio** (Long-read platform generating the full-length V1–V9 16S amplicon reads that feed the OTU clustering and reference-alignment species assignment)
- **ruffus (CGAT framework)** (Python pipeline framework organising the figure-specific analysis code, including the pipelines underlying Fig. 3 comparisons) — https://github.com/cgat-developers/ruffus
- **weinstock_full_length_16s** (Depository of the analysis code and preprocessed stool amplicon read sets used in the workflow) — https://github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Evaluation signals

- Full-length (V1–V9) 99% OTU-based Bacteroides species relative abundances provide reasonable estimates of the mWGS-measured species abundances
- Individuals group concordantly as high- vs low-Bacteroides between full-length 16S and mWGS quantification
- Bacteroides intestinalis is detected in the full-length and mWGS profiles where abundant
- B. intestinalis is absent from the V1–V3 MiSeq-derived species profiles for the sample where mWGS detects it
- Every Bacteroides OTU representative receives a species-level assignment from the RTG reference alignment before abundance aggregation

## Limitations

- V1–V3 partial 16S sequencing failed to detect the abundant Bacteroides intestinalis in one stool sample, so species assignments from partial regions can miss species
- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences
- The 99% identity threshold's ability to separate intragenomic vs inter-genomic variation was not determined for all bacterial species
- Ignoring insertions/deletions in intragenomic copy polymorphisms is imperfect — a single deletion in one of the seven E. coli K-12 MG1655 16S genes demonstrates this

## Evidence

- [abstract] Full-length sequencing resolves substitutions but not indels between intragenomic copies: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] 99% clustering of full-length 16S gives reasonable Bacteroides species abundance estimates: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [readme] Analysis code is organised as ruffus pipelines per figure: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
- [abstract] Short-read partial 16S cannot match full-gene taxonomic resolution: "We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene."
