---
name: full-length-16s-sequencing-analysis
description: Use when amplicon FASTQ/BAM read sets from full-length (~1500 bp V1–V9)
  or partial (V1–V3) 16S sequencing must be resolved to genus/species relative abundances,
  especially for Bacteroides, and compared against whole-genome shotgun profiles.
when_to_use_negative:
- Input is already a processed OTU/feature table — this skill operates on preprocessed
  amplicon read sets and does not recluster tables
- Only a short-read V4-style sub-region is available and the goal is maximal taxonomic
  accuracy — the article shows no partial region recaptured the diversity of the full
  ~1500 bp gene
- The goal is resolving intragenomic 16S copy variants via SNP phasing/substitution
  profiles — that requires a different analysis (phasing of 16S SNPs), not OTU clustering
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: USEARCH
  role: Dereplicate and cluster 16S reads into OTUs at 99% identity, and align Bacteroides
    OTU representative sequences to the RTG reference 16S database for species-level
    taxonomy assignment
- name: PacBio
  role: Long-read platform used to sequence the full-length V1–V9 (~1500 bp) 16S amplicons
    that support species-level resolution
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the analysis pipelines pertaining
    to different figures (Figures 1–3 code in Python 2.7)
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:872191cedd8289322e8b7cd25d5095db875b79e369db94743ce9377dd29434d5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - USEARCH
  - ruffus
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# full-length-16s-sequencing-analysis

## Summary

Species-level microbiome profiling via dereplication and 99%-identity OTU clustering of full-length (V1–V9, PacBio) and partial (V1–V3, MiSeq) 16S amplicon reads, with USEARCH-based taxonomy assignment against the RTG 16S reference and validation against matched metagenomic (mWGS) quantification.

## When to use

Use when amplicon FASTQ/BAM read sets from full-length (~1500 bp V1–V9) or partial (V1–V3) 16S sequencing must be resolved to genus/species relative abundances, especially for Bacteroides, and compared against whole-genome shotgun profiles.

## When NOT to use

- Input is already a processed OTU/feature table — this skill operates on preprocessed amplicon read sets and does not recluster tables
- Only a short-read V4-style sub-region is available and the goal is maximal taxonomic accuracy — the article shows no partial region recaptured the diversity of the full ~1500 bp gene
- The goal is resolving intragenomic 16S copy variants via SNP phasing/substitution profiles — that requires a different analysis (phasing of 16S SNPs), not OTU clustering

## Inputs

- Preprocessed full-length 16S amplicon read sets (V1–V9, PacBio) for human stool samples
- Preprocessed partial 16S amplicon read sets (V1–V3, MiSeq) for the same samples
- Matched metagenomic whole-genome-shotgun (mWGS) species abundance profiles per individual
- RTG reference 16S database for species-level taxonomy assignment

## Outputs

- 99%-identity OTU tables per sequencing region/platform
- Bacteroides genus- and species-level relative abundance profiles for each 16S dataset
- Side-by-side 16S-vs-mWGS abundance comparisons (Fig. 3a–b style) including high- vs low-Bacteroides individual grouping

## How to apply

Load preprocessed stool amplicon read sets (PacBio V1–V9 full-length; MiSeq V1–V3) from github.com/TheJacksonLaboratory/weinstock_full_length_16s and dereplicate each platform's reads. Cluster each read set into OTUs at 99% sequence identity with USEARCH — the 99% threshold is the paper's cutoff chosen so full-length clusters track Bacteroides species. Align Bacteroides OTU representative sequences to the RTG reference 16S database with USEARCH to assign species-level taxonomy. Compute genus- and species-level relative abundance profiles from the OTU tables per region/platform, then compare 16S-derived abundances with matched mWGS quantification per individual (Fig. 3a–b comparisons). The rationale is that short reads of variable regions cannot match full-gene taxonomic accuracy, and that analysis should focus on substitutions while ignoring insertion/deletion-driven intragenomic 16S copy polymorphisms (a known PacBio CCS homopolymer-associated error mode). Pipelines are orchestrated with the CGAT ruffus framework; code for Figures 1–3 runs under Python 2.7.

## Related tools

- **USEARCH** (Dereplicate and cluster 16S reads into OTUs at 99% identity, and align Bacteroides OTU representative sequences to the RTG reference 16S database for species-level taxonomy assignment)
- **PacBio** (Long-read platform used to sequence the full-length V1–V9 (~1500 bp) 16S amplicons that support species-level resolution)
- **ruffus** (CGAT ruffus pipeline framework used to organize the analysis pipelines pertaining to different figures (Figures 1–3 code in Python 2.7)) — https://github.com/cgat-developers/ruffus

## Examples

```
from ruffus import pipeline_run; pipeline_run([pipeline_blast_16s], verbose=3)
```

## Evaluation signals

- Full-length (V1–V9) 99%-OTU-based Bacteroides species relative abundances provide reasonable estimates of the mWGS-measured species abundances
- Individuals group concordantly as high- vs low-Bacteroides between full-length 16S profiles and mWGS
- V1–V3 profiles fail to detect the abundant Bacteroides intestinalis seen in mWGS and full-length data in one sample
- All OTUs are clustered at exactly 99% sequence identity with USEARCH and representatives are assigned taxonomy via the RTG reference 16S database
- Genus- and species-level abundance tables are produced separately for each 16S region/platform (V1–V9 and V1–V3) before mWGS comparison

## Limitations

- V1–V3 (partial-region) sequencing failed to detect the abundant Bacteroides intestinalis in one sample, so partial-region results should be treated cautiously
- PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences; the workaround of ignoring insertions/deletions is imperfect (a single deletion occurs in one of seven E. coli K-12 MG1655 16S genes)
- The study stopped short of determining how well the 99% identity threshold separates intragenomic vs. inter-genomic variation for all bacterial species
- The Figures 1–3 analysis code is written in Python 2.7, which may require an older environment

## Evidence

- [discussion] Full-length sequencing resolves species-level Bacteroides at 99% OTUs: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [abstract] Intragenomic variants and substitution focus: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [abstract] Short reads cannot match full-gene resolution: [evidence span withheld — anchor status: not_found]
- [readme] Pipeline organization in repository: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
