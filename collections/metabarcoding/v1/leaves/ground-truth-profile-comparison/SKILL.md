---
name: ground-truth-profile-comparison
description: Use when profiling synthetic/mock metagenomes whose per-genome (SGB)
  abundance compositions are known, and you need to quantify detection and quantification
  accuracy of a profiler such as MetaPhlAn 4 with per-SGB F1 scores.
when_to_use_negative:
- The input is real (non-synthetic) metagenome data with no known ground-truth composition
  — F1 against expected SGBs cannot be computed
- You already have a per-sample relative abundance table and no reference genome compositions
  to compare against
- The goal is functional profiling (genes/pathways) rather than taxonomic detection/quantification
  accuracy
edam_operation: http://edamontology.org/operation_0305
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3318
tools:
- name: MetaPhlAn 4
  role: Clade-specific marker-gene profiler run on all 70 evaluation metagenomes to
    produce the predicted SGB-level relative abundance profiles that are scored against
    ground truth
  repo: https://github.com/biobakery/MetaPhlAn
- name: conda
  role: Installation channel used to provision MetaPhlAn for the evaluation runs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:1842e08eb2170f75698ad2851333cabbc45fea2c246daff7aa538822d7c99e33
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - conda
  edam_operation: http://edamontology.org/operation_0305
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# ground-truth-profile-comparison

## Summary

Scores a taxonomic profiler's species-level output against known ground-truth genome compositions by computing per-SGB F1 scores that combine detection (presence/absence) and quantification (relative-abundance agreement). This is the evaluation strategy MetaPhlAn 4 used to demonstrate that uSGBs are detected as accurately as kSGBs.

## When to use

Use when profiling synthetic/mock metagenomes whose per-genome (SGB) abundance compositions are known, and you need to quantify detection and quantification accuracy of a profiler such as MetaPhlAn 4 with per-SGB F1 scores.

## When NOT to use

- The input is real (non-synthetic) metagenome data with no known ground-truth composition — F1 against expected SGBs cannot be computed
- You already have a per-sample relative abundance table and no reference genome compositions to compare against
- The goal is functional profiling (genes/pathways) rather than taxonomic detection/quantification accuracy

## Inputs

- 65 synthetic metagenomes assembled from environment-matched kSGBs/uSGBs with ground-truth per-SGB abundance compositions
- 5 mixed metagenomes constructed from MAGs excluded from the MetaPhlAn 4 reference database, with expected SGB assignments
- MetaPhlAn 4 species-level (SGB-level) relative abundance profiles per sample

## Outputs

- f1_scores.csv with one F1 row per evaluated SGB (detection + quantification combined)
- Per-category aggregated F1 values as mean ± standard deviation (uSGB, kSGB, mixed-evaluation MAGs)
- Category-level comparison table showing uSGB accuracy on par with kSGB accuracy

## How to apply

Load the evaluation metagenomes together with their ground-truth per-SGB abundance compositions — here 65 synthetic metagenomes assembled from environment-matched kSGBs/uSGBs plus 5 mixed metagenomes built from MAGs excluded from the reference database. Run MetaPhlAn 4 (clade-specific marker-gene profiler) on all 70 metagenomes to produce species-level (SGB-level) relative abundance profiles, then match predicted SGBs against ground-truth compositions, treating the database-excluded MAGs in the mixed samples as their expected SGB assignments. For each SGB compute an F1 score combining detection performance (presence/absence) with quantification performance (estimated vs. expected relative abundance). Aggregate per genome category (kSGB, uSGB, mixed-evaluation MAGs) as mean ± standard deviation and check against expected anchor ranges: uSGB 0.97±0.02, kSGB 0.96±0.024, mixed-evaluation 0.98±0.006. Emit one F1 row per evaluated SGB (f1_scores.csv) so every score is traceable to a genome.

## Related tools

- **MetaPhlAn 4** (Clade-specific marker-gene profiler run on all 70 evaluation metagenomes to produce the predicted SGB-level relative abundance profiles that are scored against ground truth) — https://github.com/biobakery/MetaPhlAn
- **conda** (Installation channel used to provision MetaPhlAn for the evaluation runs)

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- Computed per-category mean F1 values fall within the reported anchor ranges: uSGB 0.97±0.02, kSGB 0.96±0.024, mixed-evaluation 0.98±0.006
- f1_scores.csv contains exactly one F1 row per evaluated SGB
- uSGB F1 is on par with kSGB F1 (within the overlapping error bars), demonstrating no accuracy penalty for uncharacterized species
- Database-excluded MAGs in the 5 mixed metagenomes are matched to their expected SGB assignments rather than left unmatched
- Each F1 row traces to a specific ground-truth SGB in a specific sample (65 synthetic + 5 mixed = 70 metagenomes evaluated)

## Limitations

- Even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats, so synthetic benchmarks drawn from well-characterized environments may overestimate accuracy elsewhere
- Environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured by current uSGBs, so ground-truth evaluations only cover genomes representable in the SGB/marker set
- The comparison evaluates taxonomic profiling only; current methods do not extensively incorporate viral or eukaryotic microbial sequences

## Evidence

- [abstract] MetaPhlAn 4 is more accurate than available alternatives on synthetic evaluations while reliably quantifying organisms with no cultured isolates.: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [abstract] Marker-gene coverage basis for the SGB-level ground-truth comparison.: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [readme] Profiler definition grounding what output is compared against ground truth.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] Known limitation framing the boundary conditions of ground-truth SGB evaluations.: "Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs"
