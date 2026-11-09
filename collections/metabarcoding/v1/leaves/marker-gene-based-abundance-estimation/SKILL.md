---
name: marker-gene-based-abundance-estimation
description: Use when profiling bacteria, archaea, eukaryotes, or viruses from metagenomic
  shotgun sequencing data, especially when accurate quantification of uncharacterized
  species (uSGBs/MAGs without cultured isolates) is required.
when_to_use_negative:
- Profiling under-characterized environmental or nonhost-associated microbial communities,
  which remain highly enriched for sequences not captured even by current uSGBs
- Extensive viral or eukaryotic microbial profiling, since current methods do not
  extensively incorporate viral or eukaryotic sequences due to their unique genomic
  architectures and quality control requirements
- Requiring reads to be assigned to a functional or full-genome reference rather than
  clade-specific markers (the profiler explains ~20% more reads in human gut but is
  marker-based, not whole-genome)
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaPhlAn 4
  role: Clade-specific marker-gene profiler run on each metagenome to produce species-level
    (SGB-level) relative abundance profiles for kSGBs and uSGBs
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn via bioconda
- name: PhyloPhlAn
  role: Provides specific workflows for phylogenetic characterization of profiled
    taxa
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:faad9344b616dcddd53412170c45e4f3ce1a8f49f6bbafcb2ec151b0935af102
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - PhyloPhlAn
  - conda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# marker-gene-based-abundance-estimation

## Summary

Profile species-level relative abundances from shotgun metagenomes using clade-specific marker genes with MetaPhlAn 4, whose marker set covers 26,970 species-level genome bins (SGBs) including 4,992 taxonomically unidentified uSGBs from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes.

## When to use

Use when profiling bacteria, archaea, eukaryotes, or viruses from metagenomic shotgun sequencing data, especially when accurate quantification of uncharacterized species (uSGBs/MAGs without cultured isolates) is required.

## When NOT to use

- Profiling under-characterized environmental or nonhost-associated microbial communities, which remain highly enriched for sequences not captured even by current uSGBs
- Extensive viral or eukaryotic microbial profiling, since current methods do not extensively incorporate viral or eukaryotic sequences due to their unique genomic architectures and quality control requirements
- Requiring reads to be assigned to a functional or full-genome reference rather than clade-specific markers (the profiler explains ~20% more reads in human gut but is marker-based, not whole-genome)

## Inputs

- metagenomic shotgun sequencing reads (FASTQ) from synthetic or real samples
- ground-truth per-SGB abundance compositions of synthetic/metagenome-assembled communities (for validation workflows)

## Outputs

- species-level (SGB-level) relative abundance profiles per sample
- per-SGB F1 scores (e.g. f1_scores.csv with one F1 row per evaluated SGB)

## How to apply

Install MetaPhlAn 4 via conda (`conda install -c bioconda metaphlan`) and run its clade-specific marker-gene profiler on each metagenome to produce species-level (SGB-level) relative abundance profiles. The marker set is defined from 1.01 M prokaryotic reference and MAG genomes, so both known species (kSGBs) and uncharacterized species (uSGBs) can be detected and quantified. To validate quantification accuracy, match predicted SGBs against ground-truth genome compositions of synthetic metagenomes, treating database-excluded MAGs in mixed samples as their expected SGB assignments, then compute per-SGB F1 scores combining detection (presence/absence) and quantification (estimated vs expected relative abundance). Aggregate F1 scores per genome category (kSGB, uSGB, mixed-evaluation MAGs); expected values are uSGB 0.97±0.02, kSGB 0.96±0.024, mixed-evaluation 0.98±0.006. For strain-level follow-up, StrainPhlAn within MetaPhlAn supports strain-level microbial population genomics.

## Related tools

- **MetaPhlAn 4** (Clade-specific marker-gene profiler run on each metagenome to produce species-level (SGB-level) relative abundance profiles for kSGBs and uSGBs) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn via bioconda)
- **PhyloPhlAn** (Provides specific workflows for phylogenetic characterization of profiled taxa)

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- Computed per-category mean F1 values fall within the reported anchor ranges: uSGB 0.97±0.02, kSGB 0.96±0.024, mixed-evaluation 0.98±0.006
- f1_scores.csv contains one F1 row per evaluated SGB
- Predicted SGBs match ground-truth genome compositions of the 65 synthetic and 5 mixed new-MAG metagenomes, with database-excluded MAGs treated as expected SGB assignments
- MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome

## Limitations

- Further work remains to better profile under-characterized habitats; environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements
- Tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge
- Further integration of isolate genomes and new methods for defining taxonomic clades above the microbial family level are still needed

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [results] MetaPhlAn 4 detected and quantified uSGBs with high F1 on synthetic communities: "MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome"
- [intro] Installation via conda: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [intro] MetaPhlAn is a species-level profiling tool from shotgun data: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] Under-characterized habitats remain poorly captured by uSGBs: "Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs"
