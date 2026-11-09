---
name: synthetic-community-benchmarking
description: Use when you need to quantify how accurately a profiler (e.g., MetaPhlAn
  4) detects and quantifies organisms — including uncharacterized species (uSGBs)
  — in samples where the true genome composition is known by construction.
when_to_use_negative:
- Evaluating profiling on real biological samples where no ground-truth genome composition
  exists — the F1 computation requires known expected abundances
- Benchmarking functional profiling (e.g., HUMAnN pathway quantification) rather than
  taxonomic/species-level abundance detection
- Datasets dominated by viral or eukaryotic sequences, which the article notes current
  methods do not extensively incorporate
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaPhlAn 4
  role: Clade-specific marker-gene-based profiler run on each of the 70 synthetic/mixed
    metagenomes to produce the SGB-level abundance profiles being benchmarked
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel for MetaPhlAn (`conda install -c bioconda metaphlan`)
    used to set up the benchmarking environment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:1795fe8a49a01209668c2ffbb91507b63e3cbf6660d191eb4f399ec9cb372cf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - conda
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# synthetic-community-benchmarking

## Summary

Benchmark a taxonomic profiler's detection and quantification accuracy against ground truth using synthetic metagenomes with known genome compositions, computing per-genome F1 scores that combine presence/absence detection with relative-abundance quantification.

## When to use

Use when you need to quantify how accurately a profiler (e.g., MetaPhlAn 4) detects and quantifies organisms — including uncharacterized species (uSGBs) — in samples where the true genome composition is known by construction.

## When NOT to use

- Evaluating profiling on real biological samples where no ground-truth genome composition exists — the F1 computation requires known expected abundances
- Benchmarking functional profiling (e.g., HUMAnN pathway quantification) rather than taxonomic/species-level abundance detection
- Datasets dominated by viral or eukaryotic sequences, which the article notes current methods do not extensively incorporate

## Inputs

- 65 synthetic metagenomes assembled from environment-matched kSGBs and uSGBs (shotgun sequencing reads)
- 5 mixed metagenomes constructed from MAGs excluded from the reference database
- Ground-truth per-SGB abundance composition tables for all 70 metagenomes
- Expected SGB assignments for the database-excluded MAGs used in mixed samples

## Outputs

- Species-level (SGB-level) relative abundance profile per metagenome from MetaPhlAn 4
- f1_scores.csv containing one F1 row per evaluated SGB
- Per-category (kSGB, uSGB, mixed-evaluation MAG) mean and standard deviation of F1 scores

## How to apply

Assemble synthetic metagenomes from environment-matched genomes of known categories (kSGBs and uSGBs), plus mixed metagenomes built from MAGs deliberately excluded from the profiler's reference database, recording the ground-truth per-SGB abundance composition. Run the profiler under test (MetaPhlAn 4, a clade-specific marker-gene-based profiler installed via conda) on each metagenome to produce species-level (SGB-level) relative abundance profiles. Match predicted SGBs against ground truth, treating database-excluded MAGs in mixed metagenomes as their expected SGB assignments. Compute per-SGB F1 scores that jointly capture detection (presence/absence) and quantification (estimated vs. expected relative abundance), then aggregate per genome category as mean +/- standard deviation. Correct application is verified when per-category means land within the reported anchor ranges: uSGB 0.97+/-0.02, kSGB 0.96+/-0.024, mixed-evaluation MAGs 0.98+/-0.006.

## Related tools

- **MetaPhlAn 4** (Clade-specific marker-gene-based profiler run on each of the 70 synthetic/mixed metagenomes to produce the SGB-level abundance profiles being benchmarked) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel for MetaPhlAn (`conda install -c bioconda metaphlan`) used to set up the benchmarking environment)

## Examples

```
conda install -c bioconda metaphlan; metaphlan sample_01.fastq --input_type fastq --output_file sample_01_profile.txt
```

## Evaluation signals

- Computed per-category mean F1 falls within the reported anchors: uSGB 0.97+/-0.02, kSGB 0.96+/-0.024, mixed-evaluation 0.98+/-0.006
- f1_scores.csv contains exactly one F1 row per evaluated SGB (no missing or duplicated SGB entries)
- uSGB F1 is on par with kSGB F1 (difference within combined standard deviations), demonstrating reliable quantification of organisms with no cultured isolates
- Mixed-evaluation MAGs (excluded from the database) still yield mean F1 ~0.98, confirming expected SGB assignments were handled correctly
- Per-sample profiles cover all 70 metagenomes (65 synthetic + 5 mixed) with relative abundances comparable to the ground-truth compositions

## Limitations

- Ground truth is only available by construction; results reflect synthetic community complexity and may not transfer to real, more diverse communities
- Even with the extended MetaPhlAn 4 SGB and marker set, under-characterized habitats remain hard to profile; environmental and nonhost-associated communities are still highly enriched for sequences not captured by current uSGBs
- A sensitivity/specificity tradeoff when applying quality control to identify real but rare taxa remains an unaddressed downstream statistical challenge

## Evidence

- [results] On 65 synthetic metagenomes, MetaPhlAn 4 detected and quantified uSGBs with an average F1 score of 0.97 ± 0.02, on par with known species (kSGBs).: [evidence span withheld — anchor status: not_found]
- [abstract] proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [intro] MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [readme] To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
