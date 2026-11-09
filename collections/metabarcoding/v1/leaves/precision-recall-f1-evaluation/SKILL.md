---
name: precision-recall-f1-evaluation
description: Use when comparing taxonomic profiling tools (e.g. MetaPhlAn 4 vs MetaPhlAn
  3, mOTUs 2.6, Bracken 2.5) on synthetic metagenomes such as CAMI 2 or SynPhlAn where
  the ground-truth species composition is known.
when_to_use_negative:
- No ground-truth species composition is available (e.g. real environmental or host-associated
  metagenomes) — OPAL metrics like precision, recall and F1 cannot be computed
- The task is functional profiling (e.g. HUMAnN-style pathway quantification) rather
  than species-level taxonomic profiling
- Input samples are the under-characterized environments the paper flags as still
  'highly enriched for sequences not captured even by current uSGBs', where ground-truth-based
  evaluation of known species is not meaningful
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0203
tools:
- name: OPAL
  role: Evaluation framework that takes predicted profiles plus synthetic ground truth
    and computes precision, recall, F1, Bray-Curtis dissimilarity, RMSE, and TP/FP
    counts
- name: MetaPhlAn 4
  role: Marker-based species-level profiler under evaluation, using the SGB marker
    database covering 26,970 species-level genome bins
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn 3
  role: Baseline profiler run under default settings to produce comparable species-level
    profiles for the benchmark
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:e6e03a36d2f23ca0f9364926d29fa6a410f29158ddd8f61d43415c0cc18779c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 3
  - MetaPhlAn 4
  - OPAL
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# precision-recall-f1-evaluation

## Summary

Benchmark species-level taxonomic profilers by feeding each predicted profile plus the synthetic-community ground truth into the OPAL framework to compute per-sample precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, and true/false-positive counts, then aggregate metrics as mean ± standard deviation across samples.

## When to use

Use when comparing taxonomic profiling tools (e.g. MetaPhlAn 4 vs MetaPhlAn 3, mOTUs 2.6, Bracken 2.5) on synthetic metagenomes such as CAMI 2 or SynPhlAn where the ground-truth species composition is known.

## When NOT to use

- No ground-truth species composition is available (e.g. real environmental or host-associated metagenomes) — OPAL metrics like precision, recall and F1 cannot be computed
- The task is functional profiling (e.g. HUMAnN-style pathway quantification) rather than species-level taxonomic profiling
- Input samples are the under-characterized environments the paper flags as still 'highly enriched for sequences not captured even by current uSGBs', where ground-truth-based evaluation of known species is not meaningful

## Inputs

- Species-relative-abundance profiles from MetaPhlAn 4, MetaPhlAn 3, mOTUs 2.6, and Bracken 2.5 (one per sample)
- Synthetic community ground-truth composition files for the 128 CAMI 2 and 5 SynPhlAn-nonhuman synthetic metagenomes

## Outputs

- Per-sample OPAL metrics: precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, true/false-positive counts
- Aggregated benchmark table opal_benchmark_metrics.csv with F1, Bray-Curtis, RMSE, TP and FP fields for all four profilers

## How to apply

Run each profiler (MetaPhlAn 4 with its SGB marker database covering 26,970 species-level genome bins, plus baselines under default settings) on the 128 CAMI 2 and 5 SynPhlAn synthetic metagenomes to produce species-relative-abundance profiles. Feed each predicted profile together with the synthetic community ground truth into the OPAL framework to compute per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE on relative abundances, and true/false-positive counts. Aggregate the OPAL metrics per profiler across all synthetic samples as mean ± standard deviation. Report anchors such as MetaPhlAn 4 average Bray-Curtis 0.13±0.07 and RMSE 0.016±0.019 versus MetaPhlAn 3 (avg. BC 0.19±0.12; avg. RMSE 0.019±0.018), and write the aggregated comparison to opal_benchmark_metrics.csv.

## Related tools

- **OPAL** (Evaluation framework that takes predicted profiles plus synthetic ground truth and computes precision, recall, F1, Bray-Curtis dissimilarity, RMSE, and TP/FP counts)
- **MetaPhlAn 4** (Marker-based species-level profiler under evaluation, using the SGB marker database covering 26,970 species-level genome bins) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn 3** (Baseline profiler run under default settings to produce comparable species-level profiles for the benchmark) — https://github.com/biobakery/MetaPhlAn

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- opal_benchmark_metrics.csv exists and contains F1, Bray-Curtis, RMSE, TP and FP fields for all four profilers
- MetaPhlAn 4 mean values fall within one standard deviation of the reported anchors (Bray-Curtis 0.13±0.07, RMSE 0.016±0.019)
- MetaPhlAn 4 true positives (96.65±66.08) and false positives (16.09±17.65) are reproduced against the MetaPhlAn 3 baseline (85.32±61.95 true positives)
- All 133 synthetic samples (128 CAMI 2 + 5 SynPhlAn) have per-sample OPAL metrics for every profiler
- Aggregated metrics are reported as mean ± standard deviation per profiler across all synthetic samples

## Limitations

- Synthetic-community evaluation only measures performance against known species; even the extended MetaPhlAn 4 SGB and marker set leaves 'further work ... to better profile under-characterized habitats'
- A downstream 'tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge'
- MetaPhlAn's marker-based design means evaluation metrics reflect species-level profiling of bacteria/archaea rather than viral or eukaryotic microbial sequences, which current methods 'do not extensively incorporate'

## Evidence

- [supplementary] Feed each predicted profile together with the synthetic community ground truth into the OPAL framework to compute per-sample species-level precision, recall, F1, Bray-Curtis dissimilarity, RMSE: [evidence span withheld — anchor status: not_found]
- [results] MetaPhlAn 4 outperformed the alternative methods in abundance quantification, achieving an average Bray-Curtis dissimilarity of 0.13 ± 0.07 and RMSE of 0.016 ± 0.019: [evidence span withheld — anchor status: not_found]
- [abstract] proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates: "proves more accurate than available alternatives on synthetic evaluations while also reliably quantifying organisms with no cultured isolates"
- [readme] MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
