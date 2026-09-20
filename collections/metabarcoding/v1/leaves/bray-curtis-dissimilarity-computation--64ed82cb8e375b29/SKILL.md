---
name: bray-curtis-dissimilarity-computation
description: Use when comparing predicted species-level taxonomic abundance profiles
  against a known ground-truth community composition and you need a composition-aware
  distance metric for abundance quantification error.
when_to_use_negative:
- Input profiles are from real (non-synthetic) metagenomes with no ground-truth community
  composition, so no reference exists for the dissimilarity
- The question is species detection (presence/absence) rather than abundance quantification
  — precision, recall, F1, and TP/FP counts are the appropriate OPAL metrics
- Input is raw sequencing data rather than an already-computed abundance profile;
  run the profiler first
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaPhlAn 4
  role: Produces the species-relative-abundance profiles (marker-based, SGB marker
    database covering 26,970 species-level genome bins) whose abundances are scored
    against ground truth with Bray-Curtis dissimilarity
  repo: https://github.com/biobakery/MetaPhlAn
- name: OPAL
  role: Benchmark framework that computes per-sample species-level Bray-Curtis dissimilarity
    (plus precision, recall, F1, RMSE, TP/FP) from predicted profiles vs. synthetic
    ground truth
- name: MetaPhlAn 3
  role: Baseline profiler producing comparable species-level profiles whose abundance
    quantification error is measured with the same Bray-Curtis metric (avg. BC 0.19
    ± 0.12)
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:c55409c3e584a694dbb0c21db57e479b475453f98c8df9684f4842e0eaa29f76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 3
  - MetaPhlAn 4
  - OPAL
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# bray-curtis-dissimilarity-computation

## Summary

Computation of per-sample Bray-Curtis dissimilarity between predicted species-level relative-abundance profiles and synthetic-community ground truth, used in the OPAL benchmark to quantify abundance-quantification accuracy of taxonomic profilers such as MetaPhlAn 4.

## When to use

Use when comparing predicted species-level taxonomic abundance profiles against a known ground-truth community composition and you need a composition-aware distance metric for abundance quantification error.

## When NOT to use

- Input profiles are from real (non-synthetic) metagenomes with no ground-truth community composition, so no reference exists for the dissimilarity
- The question is species detection (presence/absence) rather than abundance quantification — precision, recall, F1, and TP/FP counts are the appropriate OPAL metrics
- Input is raw sequencing data rather than an already-computed abundance profile; run the profiler first

## Inputs

- Predicted species-level relative-abundance profile (MetaPhlAn 4 or baseline profiler output)
- Synthetic community ground-truth species-relative-abundance composition
- Set of 128 CAMI 2 and 5 SynPhlAn synthetic metagenomes

## Outputs

- Per-sample species-level Bray-Curtis dissimilarity values
- Per-profiler aggregated Bray-Curtis mean ± standard deviation across all synthetic samples (written to opal_benchmark_metrics.csv)

## How to apply

Feed each profiler's predicted species-relative-abundance profile together with the synthetic community ground truth into the OPAL framework, which computes per-sample species-level Bray-Curtis dissimilarity alongside precision, recall, F1, and RMSE on relative abundances. Apply this to each of the 128 CAMI 2 and 5 SynPhlAn synthetic metagenomes profiled by MetaPhlAn 4 and the baselines (MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) under default settings. Aggregate the per-sample Bray-Curtis values per profiler across all synthetic samples as mean ± standard deviation. Rationale: Bray-Curtis captures differences in relative abundances (not just presence/absence), so a profiler with accurate abundance quantification such as MetaPhlAn 4 achieves a lower dissimilarity (avg. 0.13 ± 0.07) than one that detects similar taxa but mis-quantifies them. Pair Bray-Curtis with TP/FP counts because a low dissimilarity alone does not certify species-level precision and recall.

## Related tools

- **MetaPhlAn 4** (Produces the species-relative-abundance profiles (marker-based, SGB marker database covering 26,970 species-level genome bins) whose abundances are scored against ground truth with Bray-Curtis dissimilarity) — https://github.com/biobakery/MetaPhlAn
- **OPAL** (Benchmark framework that computes per-sample species-level Bray-Curtis dissimilarity (plus precision, recall, F1, RMSE, TP/FP) from predicted profiles vs. synthetic ground truth)
- **MetaPhlAn 3** (Baseline profiler producing comparable species-level profiles whose abundance quantification error is measured with the same Bray-Curtis metric (avg. BC 0.19 ± 0.12)) — https://github.com/biobakery/MetaPhlAn

## Evaluation signals

- Aggregated MetaPhlAn 4 mean Bray-Curtis falls within one standard deviation of the reported anchor 0.13 ± 0.07
- opal_benchmark_metrics.csv exists and contains F1, Bray-Curtis, RMSE, TP and FP fields for all four profilers
- Bray-Curtis values are bounded in [0, 1] and equal 0 for a predicted profile identical to the ground truth
- Aggregation is mean ± standard deviation across all 133 synthetic samples (128 CAMI 2 + 5 SynPhlAn) per profiler
- Baseline comparison sanity check: MetaPhlAn 4 dissimilarity is lower than MetaPhlAn 3's (0.13 ± 0.07 vs 0.19 ± 0.12)

## Limitations

- Anchored benchmark values apply to synthetic metagenomes (CAMI 2 and SynPhlAn-nonhuman); even with the extended MetaPhlAn 4 SGB and marker set, environmental, nonhost-associated, and under-studied microbial communities remain highly enriched for sequences not captured by current uSGBs, so behavior on real nonhuman data is not covered by these anchors
- Bray-Curtis measures abundance error only; it must be reported together with precision/recall/F1 and TP/FP counts to characterize detection performance
- MetaPhlAn-based profiles are marker-based and relative-abundance only; the metric does not address viral or eukaryotic sequences that current methods do not extensively incorporate

## Evidence

- [results] On the synthetic metagenomes, MetaPhlAn 4 outperformed the alternative methods in abundance quantification, achieving an average Bray-Curtis dissimilarity of 0.13 ± 0.07 and RMSE of 0.016 ± 0.019: [evidence span withheld — anchor status: not_found]
- [methods] OPAL computes per-sample metrics: [evidence span withheld — anchor status: not_found]
- [readme] MetaPhlAn species-level profiling scope: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [abstract] SGB marker database size: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
