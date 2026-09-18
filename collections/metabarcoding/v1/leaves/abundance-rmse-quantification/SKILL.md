---
name: abundance-rmse-quantification
description: Use when comparing species-level taxonomic profilers on synthetic metagenomes
  with known community composition and you need abundance-quantification accuracy
  metrics such as Bray-Curtis dissimilarity, RMSE, F1, and TP/FP counts.
when_to_use_negative:
- Input metagenomes are real (non-synthetic) samples without a known ground-truth
  community composition — OPAL abundance-error metrics (Bray-Curtis, RMSE, TP/FP)
  require truth profiles
- The goal is strain-level population genomics (StrainPhlAn) or functional profiling
  (HUMAnN) rather than species-level abundance quantification
- Communities are dominated by viral, eukaryotic, or under-characterized environmental
  taxa not captured by current uSGBs, where the article notes profiling remains incomplete
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: MetaPhlAn 4
  role: Query profiler producing species-relative-abundance profiles from shotgun
    metagenomes via its SGB marker database
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn 3
  role: Baseline species-level profiler run under default settings on the same 133
    synthetic samples for comparison
  repo: https://github.com/biobakery/MetaPhlAn
- name: OPAL
  role: Benchmark framework computing per-sample species-level precision, recall,
    F1, Bray-Curtis dissimilarity, RMSE, and TP/FP counts against ground truth
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:274a0a3e1ee074fd112fa659d070989aedea5b50f41981271c827f01a86d28d8
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
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# abundance-rmse-quantification

## Summary

Benchmark species-level metagenomic taxonomic profilers (MetaPhlAn 4 vs MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) on synthetic metagenomes with known ground truth, using the OPAL framework to quantify abundance accuracy via Bray-Curtis dissimilarity, RMSE on relative abundances, F1, and true/false-positive counts.

## When to use

Use when comparing species-level taxonomic profilers on synthetic metagenomes with known community composition and you need abundance-quantification accuracy metrics such as Bray-Curtis dissimilarity, RMSE, F1, and TP/FP counts.

## When NOT to use

- Input metagenomes are real (non-synthetic) samples without a known ground-truth community composition — OPAL abundance-error metrics (Bray-Curtis, RMSE, TP/FP) require truth profiles
- The goal is strain-level population genomics (StrainPhlAn) or functional profiling (HUMAnN) rather than species-level abundance quantification
- Communities are dominated by viral, eukaryotic, or under-characterized environmental taxa not captured by current uSGBs, where the article notes profiling remains incomplete

## Inputs

- Synthetic shotgun metagenomes with known ground-truth species compositions (128 CAMI 2 and 5 SynPhlAn-nonhuman samples)
- Species-level relative-abundance profiles from MetaPhlAn 4 (SGB marker database, 26,970 species-level genome bins)
- Species-level profiles from baseline profilers (MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) run under default settings
- Synthetic community ground-truth abundance profiles for OPAL scoring

## Outputs

- Per-sample species-level OPAL metrics: precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, true/false-positive counts
- Aggregated benchmark table opal_benchmark_metrics.csv comparing MetaPhlAn 4 against the three baselines as mean ± standard deviation

## How to apply

Run MetaPhlAn 4 — the marker-based species-level profiler whose SGB marker database covers 26,970 species-level genome bins from ~1.01 M prokaryotic reference and metagenome-assembled genomes — on each of the 128 CAMI 2 and 5 SynPhlAn-nonhuman synthetic metagenomes to produce species-relative-abundance profiles. Run the baselines (MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) on the same 133 synthetic samples under default settings for comparable profiles. Feed each predicted profile together with the synthetic community ground truth into OPAL to compute per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, and true/false-positive counts. Aggregate the OPAL metrics per profiler across all synthetic samples as mean ± standard deviation; the reported anchors for MetaPhlAn 4 are avg Bray-Curtis 0.13 ± 0.07, RMSE 0.016 ± 0.019, 96.65 ± 66.08 true positives and 16.09 ± 17.65 false positives (vs MetaPhlAn 3 at 85.32 ± 61.95 true positives). Write the aggregated comparison to opal_benchmark_metrics.csv and check that MetaPhlAn 4 means fall within one standard deviation of the anchors.

## Related tools

- **MetaPhlAn 4** (Query profiler producing species-relative-abundance profiles from shotgun metagenomes via its SGB marker database) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn 3** (Baseline species-level profiler run under default settings on the same 133 synthetic samples for comparison) — https://github.com/biobakery/MetaPhlAn
- **OPAL** (Benchmark framework computing per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE, and TP/FP counts against ground truth)

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- opal_benchmark_metrics.csv exists and contains F1, Bray-Curtis, RMSE, TP and FP fields for all four profilers (MetaPhlAn 4, MetaPhlAn 3, mOTUs 2.6, Bracken 2.5)
- MetaPhlAn 4 mean Bray-Curtis falls within one standard deviation of the anchor 0.13 ± 0.07 and mean RMSE within one SD of 0.016 ± 0.019
- MetaPhlAn 4 mean true positives ≈ 96.65 ± 66.08 and false positives ≈ 16.09 ± 17.65, exceeding MetaPhlAn 3's 85.32 ± 61.95 true positives
- All 133 synthetic samples (128 CAMI 2 + 5 SynPhlAn-nonhuman) are represented in the aggregated metrics per profiler
- Metrics were aggregated as mean ± standard deviation across samples, matching the reported averaging scheme

## Limitations

- Further work remains to better profile under-characterized habitats; environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured by current uSGBs
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements
- A tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge

## Evidence

- [abstract] MetaPhlAn 4 tool description: "Here we present MetaPhlAn 4, which integrates information from metagenome assemblies and microbial isolate genomes for more comprehensive metagenomic taxonomic profiling."
- [abstract] Synthetic-evaluation superiority finding: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [readme] Species-level profiling scope: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
