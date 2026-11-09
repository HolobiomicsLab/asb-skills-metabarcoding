---
name: species-level-taxonomic-profiling
description: Use when metagenomic shotgun FASTQ reads must be converted into species-level
  relative-abundance profiles (bacteria, archaea, eukaryotes, viruses), including
  organisms lacking cultured isolates.
when_to_use_negative:
- When functional (gene/pathway) profiling rather than taxonomic profiling is needed
  — MetaPhlAn produces taxonomic abundances, not functional profiles (use HUMAnN from
  bioBakery)
- When strain-level population genomics rather than species-level abundance profiling
  is the goal — that is the separate StrainPhlAn workflow within MetaPhlAn
- When the community is environmental/nonhost-associated and heavily enriched for
  sequences not captured by current uSGBs — profiling coverage will be limited
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaPhlAn 4
  role: Marker-based species-level taxonomic profiler; integrates metagenome assemblies
    and isolate genomes via SGB markers (26,970 SGBs) to profile known and uncharacterized
    species
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: General computational tool for species-level microbial profiling (bacteria,
    archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data; installable
    via bioconda
  repo: https://github.com/biobakery/MetaPhlAn
- name: OPAL
  role: Benchmark framework used to compute per-sample species-level precision, recall,
    F1, Bray-Curtis dissimilarity, and RMSE of predicted profiles against synthetic
    ground truth
- name: bioBakery
  role: Integrated meta'omic analysis environment in which MetaPhlAn-based taxonomic
    profiling is deployed
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:98529ff4d8bfc6c95029f06f0aacb27f00e1c7c960803cb6ce4b2c59ac8eb628
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - OPAL
  - bioBakery
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# species-level-taxonomic-profiling

## Summary

Species-level microbial taxonomic profiling of metagenomic shotgun sequencing data using MetaPhlAn 4, a marker-based profiler whose SGB marker database covers 26,970 species-level genome bins derived from 1.01 M prokaryotic reference and metagenome-assembled genomes.

## When to use

Use when metagenomic shotgun FASTQ reads must be converted into species-level relative-abundance profiles (bacteria, archaea, eukaryotes, viruses), including organisms lacking cultured isolates.

## When NOT to use

- When functional (gene/pathway) profiling rather than taxonomic profiling is needed — MetaPhlAn produces taxonomic abundances, not functional profiles (use HUMAnN from bioBakery)
- When strain-level population genomics rather than species-level abundance profiling is the goal — that is the separate StrainPhlAn workflow within MetaPhlAn
- When the community is environmental/nonhost-associated and heavily enriched for sequences not captured by current uSGBs — profiling coverage will be limited

## Inputs

- Metagenomic shotgun sequencing reads (FASTQ) from bacterial, archaeal, eukaryotic, or viral communities
- MetaPhlAn SGB marker database (26,970 species-level genome bins, built from 1.01 M reference and metagenome-assembled genomes)
- Synthetic metagenome ground-truth profiles (e.g. CAMI 2 or SynPhlAn) when benchmarking with OPAL

## Outputs

- Per-sample species-level relative-abundance profile (MetaPhlAn output table)
- OPAL benchmark metrics per profiler (precision, recall, F1, Bray-Curtis dissimilarity, RMSE, true/false-positive counts), e.g. aggregated in opal_benchmark_metrics.csv

## How to apply

Install MetaPhlAn (e.g. via conda from bioconda) and run its marker-based profiler on each shotgun metagenome against the SGB marker database covering 26,970 species-level genome bins, which integrates unique marker genes from reference isolate genomes and metagenome-assembled genomes (1.01 M prokaryotic genomes, 4,992 taxonomically unidentified at species level). The output is a species-relative-abundance profile per sample. Because MetaPhlAn 4 includes uncharacterized species-level genome bins (uSGBs), it explains ~20% more reads in most human gut microbiomes and >40% in less-characterized environments such as the rumen, and reliably quantifies organisms with no cultured isolates. To benchmark profile accuracy, feed predicted profiles plus synthetic-community ground truth into OPAL to compute per-sample precision, recall, F1, Bray-Curtis dissimilarity, and RMSE on relative abundances; on CAMI 2 and SynPhlAn synthetic metagenomes MetaPhlAn 4 achieved avg Bray-Curtis 0.13±0.07 and RMSE 0.016±0.019, outperforming MetaPhlAn 3, mOTUs 2.6, and Bracken 2.5.

## Related tools

- **MetaPhlAn 4** (Marker-based species-level taxonomic profiler; integrates metagenome assemblies and isolate genomes via SGB markers (26,970 SGBs) to profile known and uncharacterized species) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (General computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data; installable via bioconda) — https://github.com/biobakery/MetaPhlAn
- **OPAL** (Benchmark framework used to compute per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, and RMSE of predicted profiles against synthetic ground truth)
- **bioBakery** (Integrated meta'omic analysis environment in which MetaPhlAn-based taxonomic profiling is deployed) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan; metaphlan sample_reads.fastq --input_type fastq --bowtie2db metaphlan_db --outputfile sample_profile.txt
```

## Evaluation signals

- On CAMI 2 / SynPhlAn synthetic metagenomes, MetaPhlAn 4 mean Bray-Curtis dissimilarity should fall near 0.13±0.07 and RMSE near 0.016±0.019, within one standard deviation of the reported anchors
- MetaPhlAn 4 should outperform MetaPhlAn 3 in abundance quantification (MetaPhlAn 3 avg BC 0.19±0.12; avg RMSE 0.019±0.018; 85.32±61.95 true positives vs MetaPhlAn 4 96.65±66.08 TP, 16.09±17.65 FP)
- Benchmark validation: opal_benchmark_metrics.csv exists and contains F1, Bray-Curtis, RMSE, TP and FP fields for all four profilers (MetaPhlAn 4, MetaPhlAn 3, mOTUs 2.6, Bracken 2.5)
- Read-explanatory power check: in human gut microbiomes MetaPhlAn 4 explains ~20% more reads than alternatives, and >40% in less-characterized environments such as the rumen
- Profile sanity check: relative abundances of detected species should sum to 1 per sample, and only species present in the SGB marker database can be reported

## Limitations

- Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs; further work remains to better profile under-characterized habitats
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements
- A tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge
- Further integration of isolate genomes and new methods for defining taxonomic clades above the microbial family level are still needed; the team plans at least two new MetaPhlAn databases per year to expand profilable diversity

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [abstract] MetaPhlAn explains more reads in less-characterized environments: "MetaPhlAn 4 explains ~20% more reads in most international human gut microbiomes and >40% in less-characterized environments such as the rumen microbiome"
- [abstract] MetaPhlAn 4 accuracy on synthetic evaluations: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [intro] MetaPhlAn scope as a species-level profiler: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [readme] conda installation command from README: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [other] OPAL benchmark workflow anchors on synthetic metagenomes: [evidence span withheld — anchor status: not_found]
- [discussion] uSGB coverage limitation in under-studied communities: "Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs"
