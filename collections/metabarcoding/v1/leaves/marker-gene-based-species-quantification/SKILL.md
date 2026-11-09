---
name: marker-gene-based-species-quantification
description: Use when you need species-level relative-abundance profiles and explainable-read
  fractions from FASTQ shotgun metagenomes, especially to compare marker versions
  across environments (human gut, rumen, wild mice, ocean).
when_to_use_negative:
- The input is already a species-level abundance table and no re-profiling or version
  comparison is needed
- The goal is comprehensive read-level or functional (pathway/gene) profiling — marker-gene
  profiling quantifies clade abundance, not every sequence; use functional profilers
  like HUMAnN instead
- The community is dominated by viral or eukaryotic sequences, which the current SGB/marker
  set does not extensively incorporate
edam_operation: http://edamontology.org/operation_0364
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: MetaPhlAn 4
  role: 'Primary profiler: defines unique marker genes for 26,970 SGBs and estimates
    unclassified reads for species-level abundance profiles'
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Baseline version (3.0.12) run on the same samples to compute explainable-read
    fold increases
  repo: https://github.com/biobakery/MetaPhlAn
- name: StrainPhlAn
  role: Downstream strain-level population genomics on MetaPhlAn profiles, enabling
    single-strain resolution for uncharacterized species
  repo: https://github.com/biobakery/MetaPhlAn
- name: bioBakery
  role: Suite/virtual image providing MetaPhlAn and accompanying meta'omic profiling
    tools and tutorials
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:16004f41dc1365aa7fa43aaaed9c20a7e6e2225d2262d9d9edd2f967e5e7a55c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - StrainPhlAn
  - bioBakery
  edam_operation: http://edamontology.org/operation_0364
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# marker-gene-based-species-quantification

## Summary

Quantify species-level microbial relative abundance (bacteria, archaea, eukaryotes, viruses) from shotgun metagenomes using clade-specific marker genes, as implemented in MetaPhlAn 4, whose extended SGB/marker set (26,970 SGBs incl. 4,992 unidentified uSGBs) also profiles uncharacterized taxa.

## When to use

Use when you need species-level relative-abundance profiles and explainable-read fractions from FASTQ shotgun metagenomes, especially to compare marker versions across environments (human gut, rumen, wild mice, ocean).

## When NOT to use

- The input is already a species-level abundance table and no re-profiling or version comparison is needed
- The goal is comprehensive read-level or functional (pathway/gene) profiling — marker-gene profiling quantifies clade abundance, not every sequence; use functional profilers like HUMAnN instead
- The community is dominated by viral or eukaryotic sequences, which the current SGB/marker set does not extensively incorporate

## Inputs

- Shotgun metagenomic FASTQ files (e.g. 24,515 samples from 145 public datasets)
- MetaPhlAn 4 marker database (26,970 SGBs incl. 4,992 uSGBs)
- Prior MetaPhlAn version (3.0.12) profiles as baseline for fold-increase comparison

## Outputs

- Per-sample species-level relative-abundance profile (with clade taxonomy)
- Fraction of explainable/profiled reads per sample and environment
- Per-environment fold-increase table (e.g. environment_read_explainability_summary.tsv)
- Per-sample fraction of profiled richness attributed to uSGBs (mean 23.13%, s.d. 17.89%)

## How to apply

Install MetaPhlAn 4 via bioconda and run each shotgun metagenomic sample against its marker database of 26,970 species-level genome bins (4,992 taxonomically unidentified uSGBs) curated from ~1.01 M prokaryotic reference and metagenome-assembled genomes, enabling the unclassified-reads estimation feature. Run the predecessor (e.g. MetaPhlAn 3.0.12) on the same samples to obtain baseline profiles and explainable-read fractions. For each host environment (human gut, airways, wild mice, rumen, ocean, etc.), compute the fold increase in explainable reads of MetaPhlAn 4 over MetaPhlAn 3 and the per-sample fraction of profiled richness attributed to uSGBs. Marker-gene targeting means output is clade-relative abundance rather than full read mapping, so validate by reproducing reported anchors (~20% human gut, 1.95x airways, 3.26x wild mice, 14.15x rumen, 6.65x ocean).

## Related tools

- **MetaPhlAn 4** (Primary profiler: defines unique marker genes for 26,970 SGBs and estimates unclassified reads for species-level abundance profiles) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Baseline version (3.0.12) run on the same samples to compute explainable-read fold increases) — https://github.com/biobakery/MetaPhlAn
- **StrainPhlAn** (Downstream strain-level population genomics on MetaPhlAn profiles, enabling single-strain resolution for uncharacterized species) — https://github.com/biobakery/MetaPhlAn
- **bioBakery** (Suite/virtual image providing MetaPhlAn and accompanying meta'omic profiling tools and tutorials) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan; metaphlan sample.fastq.gz --input_type fastq --output sample_profile.tsv --unclassified_estimation
```

## Evaluation signals

- Per-environment fold increases in explainable reads match reported anchors: ~20% (human gut), 1.95x (airways), 3.26x (wild mice), 14.15x (rumen), 6.65x (ocean)
- Per-sample uSGB richness fraction summary reproduces mean 23.13% with s.d. 17.89%
- Profiles contain species-level relative abundances for bacteria, archaea, eukaryotes, and viruses and sum to a valid clade-relative composition
- Rumen-like under-characterized environments show >40% read-explainability improvement, consistent with the abstract claim
- MetaPhlAn 3.0.12 and MetaPhlAn 4 are run on identical sample sets so fold increases are computed on matched samples

## Limitations

- Further work remains to better profile under-characterized habitats; environmental, nonhost-associated, and under-studied communities remain highly enriched for sequences not captured even by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences due to their unique genomic architectures and quality-control requirements
- A tradeoff between sensitivity and specificity when applying quality-control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge

## Evidence

- [abstract] MetaPhlAn 4 integrates metagenome assemblies with reference genomes for comprehensive profiling: "Here we present MetaPhlAn 4, which integrates information from metagenome assemblies and microbial isolate genomes for more comprehensive metagenomic taxonomic profiling."
- [abstract] MetaPhlAn defines markers for 26,970 SGBs including 4,992 unidentified uSGBs: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [readme] MetaPhlAn is a species-level profiler for shotgun data: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [abstract] MetaPhlAn 4 explains more reads in human gut and rumen microbiomes: "MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome"
- [readme] Bioconda installation command for MetaPhlAn: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
