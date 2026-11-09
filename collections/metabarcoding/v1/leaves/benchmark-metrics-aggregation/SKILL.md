---
name: benchmark-metrics-aggregation
description: Use when comparing species-level accuracy of taxonomic profilers (e.g.
  MetaPhlAn 4 vs MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) on synthetic metagenomes (CAMI
  2, SynPhlAn) with known ground-truth community composition.
when_to_use_negative:
- No ground-truth community composition exists (e.g. real environmental or host-associated
  metagenomes), so OPAL precision/recall/F1 and RMSE against truth cannot be computed
- The task is functional profiling (gene/pathway abundance, e.g. HUMAnN output) rather
  than species-level taxonomic abundance comparison
- Profiler outputs are already a precomputed benchmark/feature table requiring analysis
  rather than accuracy evaluation against reference profiles
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: MetaPhlAn 4
  role: Profiler under evaluation; marker-based species-level profiling with the SGB
    marker database covering 26,970 species-level genome bins
  repo: https://github.com/biobakery/MetaPhlAn
- name: OPAL
  role: Evaluation framework that computes per-sample species-level precision, recall,
    F1, Bray-Curtis dissimilarity, RMSE and true/false-positive counts from predicted
    vs ground-truth profiles
- name: MetaPhlAn 3
  role: Baseline profiler whose OPAL metrics (avg BC 0.19 ± 0.12; avg RMSE 0.019 ±
    0.018; TP 85.32 ± 61.95) serve as the comparison anchor
  repo: https://github.com/biobakery/MetaPhlAn
- name: mOTUs 2.6
  role: Baseline profiler run on the same 133 synthetic samples under default settings
- name: Bracken 2.5
  role: Baseline profiler run on the same 133 synthetic samples under default settings
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:105fc8de5c44b997f89eb8f15bce2a184bb1b56ee72c50424e768035b5c38050
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - Bracken 2.5
  - MetaPhlAn
  - MetaPhlAn 3
  - MetaPhlAn 4
  - OPAL
  - mOTUs 2.6
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# benchmark-metrics-aggregation

## Summary

Aggregates per-sample OPAL accuracy metrics (precision, recall, F1, Bray-Curtis dissimilarity, RMSE, true/false positives) for species-level taxonomic profilers benchmarked against ground-truth synthetic metagenomes, reporting each profiler as mean ± standard deviation in a single benchmark table.

## When to use

Use when comparing species-level accuracy of taxonomic profilers (e.g. MetaPhlAn 4 vs MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) on synthetic metagenomes (CAMI 2, SynPhlAn) with known ground-truth community composition.

## When NOT to use

- No ground-truth community composition exists (e.g. real environmental or host-associated metagenomes), so OPAL precision/recall/F1 and RMSE against truth cannot be computed
- The task is functional profiling (gene/pathway abundance, e.g. HUMAnN output) rather than species-level taxonomic abundance comparison
- Profiler outputs are already a precomputed benchmark/feature table requiring analysis rather than accuracy evaluation against reference profiles

## Inputs

- Species-relative-abundance profile (species-level taxonomy table) per profiler per sample, e.g. MetaPhlAn 4 output on each of 128 CAMI 2 and 5 SynPhlAn synthetic metagenomes
- Baseline profiler profiles from MetaPhlAn 3, mOTUs 2.6 and Bracken 2.5 on the same 133 synthetic samples under default settings
- Synthetic community ground-truth composition for each CAMI 2 and SynPhlAn sample

## Outputs

- Per-sample OPAL metrics: species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, true-positive and false-positive counts
- Aggregated benchmark table opal_benchmark_metrics.csv with F1, Bray-Curtis, RMSE, TP and FP fields (mean ± standard deviation) for all four profilers

## How to apply

Run each profiler (e.g. MetaPhlAn 4 with its SGB marker database covering 26,970 species-level genome bins, and baseline profilers MetaPhlAn 3, mOTUs 2.6, Bracken 2.5 under default settings) on the same 133 synthetic samples (128 CAMI 2 plus 5 SynPhlAn-nonhuman metagenomes) to produce species-relative-abundance profiles. Feed each predicted profile together with the synthetic community ground truth into the OPAL framework, which computes per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, and true/false-positive counts. Aggregate OPAL metrics per profiler across all synthetic samples as mean ± standard deviation. The rationale is that synthetic metagenomes provide exact ground truth, enabling error-controlled comparison: MetaPhlAn 4 anchors are avg Bray-Curtis 0.13 ± 0.07, RMSE 0.016 ± 0.019, 96.65 ± 66.08 true positives and 16.09 ± 17.65 false positives, versus MetaPhlAn 3 at 85.32 ± 61.95 true positives. Write the aggregated benchmark table comparing MetaPhlAn 4 against the three baselines to opal_benchmark_metrics.csv.

## Related tools

- **MetaPhlAn 4** (Profiler under evaluation; marker-based species-level profiling with the SGB marker database covering 26,970 species-level genome bins) — https://github.com/biobakery/MetaPhlAn
- **OPAL** (Evaluation framework that computes per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE and true/false-positive counts from predicted vs ground-truth profiles)
- **MetaPhlAn 3** (Baseline profiler whose OPAL metrics (avg BC 0.19 ± 0.12; avg RMSE 0.019 ± 0.018; TP 85.32 ± 61.95) serve as the comparison anchor) — https://github.com/biobakery/MetaPhlAn
- **mOTUs 2.6** (Baseline profiler run on the same 133 synthetic samples under default settings)
- **Bracken 2.5** (Baseline profiler run on the same 133 synthetic samples under default settings)

## Evaluation signals

- opal_benchmark_metrics.csv exists and contains F1, Bray-Curtis, RMSE, TP and FP fields for all four profilers
- MetaPhlAn 4 mean Bray-Curtis dissimilarity falls within one standard deviation of the reported anchor 0.13 ± 0.07
- MetaPhlAn 4 mean RMSE falls within one standard deviation of the reported anchor 0.016 ± 0.019
- MetaPhlAn 4 mean true-positive count is near 96.65 ± 66.08 and false positives near 16.09 ± 17.65, exceeding MetaPhlAn 3's 85.32 ± 61.95 true positives
- All four profilers were run on the identical sample set (133 synthetic samples: 128 CAMI 2 and 5 SynPhlAn) so metrics are directly comparable

## Limitations

- Synthetic-metagenome evaluation reflects communities with known ground truth; on real, under-characterized habitats the marker database may miss many sequences, since environmental and nonhost-associated communities are highly enriched for sequences not captured by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences, so species-level benchmark metrics cover only the profilers' bacterial/archaeal marker space
- OPAL metrics depend on the profiler's abundance quantification; profilers with different read-mapping sensitivity versus specificity tradeoffs (e.g. quality-control filtering of rare taxa) may shift TP/FP counts in ways the mean ± SD aggregation does not fully characterize

## Evidence

- [abstract] MetaPhlAn 4 is more accurate than available alternatives on synthetic evaluations while reliably quantifying organisms with no cultured isolates.: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [abstract] MetaPhlAn 4 marker database scale underlying the profiles being benchmarked.: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [readme] Profiler being evaluated is a species-level taxonomic profiler for shotgun metagenomes.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [discussion] Scope limitation on the taxa covered by current profiling methods, relevant to interpreting benchmark metrics.: "The current methods also do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements relative to bacterial and"
