---
name: species-level-detection-quantification-f1-scoring
description: Use when benchmarking a metagenomic taxonomic profiler against known
  ground-truth genome compositions, especially to verify that uncharacterized species
  (uSGBs/MAGs) are quantified as accurately as known species (kSGBs).
when_to_use_negative:
- Input samples lack ground-truth genome-level abundance compositions — F1 scoring
  against expected SGBs cannot be performed, so use read-mapping or diversity-based
  evaluation instead
- The reference database already contains all organisms under test — the mixed-MAG
  evaluation (database-excluded genomes) loses its novelty-leakage check
- The goal is strain-level or functional (pathway) profiling rather than species-level
  detection/quantification accuracy
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: MetaPhlAn 4
  role: Clade-specific marker-gene profiler run on each of the 70 metagenomes to produce
    the species-level (SGB-level) relative abundance profiles being scored
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel used to deploy MetaPhlAn for the benchmark
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:3f30cf568b80702f8f064852f36aff092a907a9c7c96c196f729ba7ee9eb7e5b
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# species-level-detection-quantification-f1-scoring

## Summary

Benchmarking protocol for evaluating whether a taxonomic profiler detects and accurately quantifies both known species (kSGBs) and uncharacterized species (uSGBs/MAGs) by computing per-genome F1 scores against ground-truth abundance compositions, as done for MetaPhlAn 4 on 65 synthetic plus 5 mixed metagenomes.

## When to use

Use when benchmarking a metagenomic taxonomic profiler against known ground-truth genome compositions, especially to verify that uncharacterized species (uSGBs/MAGs) are quantified as accurately as known species (kSGBs).

## When NOT to use

- Input samples lack ground-truth genome-level abundance compositions — F1 scoring against expected SGBs cannot be performed, so use read-mapping or diversity-based evaluation instead
- The reference database already contains all organisms under test — the mixed-MAG evaluation (database-excluded genomes) loses its novelty-leakage check
- The goal is strain-level or functional (pathway) profiling rather than species-level detection/quantification accuracy

## Inputs

- 65 synthetic metagenomes assembled from environment-matched kSGBs and uSGBs (shotgun sequencing reads)
- 5 mixed metagenomes constructed from MAGs excluded from the MetaPhlAn 4 reference database
- Ground-truth per-SGB abundance compositions for each of the 70 metagenomes
- MetaPhlAn 4 marker-gene database covering 26,970 species-level genome bins (4,992 taxonomically unidentified at species level)

## Outputs

- Species-level (SGB-level) relative abundance profiles per metagenome (MetaPhlAn 4 output)
- f1_scores.csv containing one F1 row per evaluated SGB
- Per-category aggregated F1 statistics: uSGB 0.97+/-0.02, kSGB 0.96+/-0.024, mixed-evaluation MAGs 0.98+/-0.006

## How to apply

Assemble (or obtain) synthetic metagenomes built from environment-matched kSGBs and uSGBs, plus mixed metagenomes constructed from MAGs deliberately excluded from the profiler's reference database, each with ground-truth per-SGB abundance compositions. Run MetaPhlAn 4 — the clade-specific marker-gene profiler that defines unique markers for 26,970 species-level genome bins from ~1.01 M reference and metagenome-assembled genomes — on each sample to obtain species-level (SGB-level) relative abundance profiles. Match predicted SGBs to ground truth, treating database-excluded MAGs in mixed samples as their expected SGB assignments so novel organisms are scored without database leakage. For each SGB compute an F1 score that jointly captures detection (presence/absence correctness) and quantification (estimated vs. expected relative abundance). Aggregate per genome category as mean +/- standard deviation; correct application reproduces uSGB 0.97+/-0.02, kSGB 0.96+/-0.024, and mixed-evaluation MAG 0.98+/-0.006.

## Related tools

- **MetaPhlAn 4** (Clade-specific marker-gene profiler run on each of the 70 metagenomes to produce the species-level (SGB-level) relative abundance profiles being scored) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel used to deploy MetaPhlAn for the benchmark) — https://github.com/biobakery/MetaPhlAn

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- Aggregated per-category F1 means fall within the anchor ranges: uSGB 0.97+/-0.02, kSGB 0.96+/-0.024, mixed-evaluation MAGs 0.98+/-0.006
- f1_scores.csv contains exactly one F1 row per evaluated SGB across the 70 metagenomes
- uSGB F1 scores are on par with kSGB F1 scores (difference within the reported standard deviations), indicating no accuracy penalty for uncharacterized species
- Predicted SGBs in the mixed metagenomes match the expected SGB assignments of the deliberately database-excluded MAGs
- Each MetaPhlAn run yields a valid species-level relative abundance profile (SGB-level abundances) for every input sample

## Limitations

- MetaPhlAn 4's extended SGB and marker set still leaves under-characterized habitats poorly profiled; environmental, nonhost-associated, and under-studied communities remain highly enriched for sequences not captured by current uSGBs
- The current method does not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements
- Synthetic-community F1 performance may not transfer to real communities, where a sensitivity/specificity tradeoff under quality control of rare taxa remains an unaddressed challenge

## Evidence

- [abstract] synthetic evaluation anchor: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [abstract] database scope: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [intro] profiler definition: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [readme] installation route: "To install MetaPhlan through conda run:

$ conda install -c bioconda metaphlan"
