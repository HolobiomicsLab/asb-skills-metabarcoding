---
name: taxonomic-profiler-benchmark-visualization
description: Use when you have a per-pairing benchmark metrics table (sensitivity,
  specificity, error score, alpha diversity deviation for each profiler-library combination)
  and need to determine which metrics are correlated and which pairings occupy the
  high-sensitivity/high-specificity quadrant.
when_to_use_negative:
- Input is raw sequencing data or taxonomy calls rather than an already-computed benchmark
  metrics table — profiling must be run first (e.g., with PathoScope 2 or Kraken 2)
  before correlations can be computed
- The analysis goal is per-tool accuracy ranking alone (e.g., which library is most
  accurate) without examining cross-metric relationships — use a direct benchmark
  comparison instead
- The data are single-sample taxonomic profiles rather than aggregated mock-community
  benchmark metrics across the 136-sample mock community set
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3318
tools:
- name: DADA2
  role: Benchmark profiler whose genus/species sensitivity, specificity, error score,
    and alpha diversity deviation metrics feed the correlation analysis and scatterplots
- name: QIIME 2
  role: Benchmark profiler (DADA2 plugin) contributing per-pairing metric rows to
    the metrics table being visualized
- name: Mothur
  role: Benchmark profiler contributing per-pairing metric rows to the metrics table
    being visualized
- name: PathoScope 2
  role: Benchmark profiler that dominates the high-sensitivity/high-specificity upper
    right quadrant; its per-pairing metrics are plotted and correlated
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Benchmark profiler (with Standard library) dominating the high-sensitivity/high-specificity
    quadrant; its per-pairing metrics are plotted and correlated
  repo: github.com/aubreyodom/16SBenchmarking
- name: scipy.stats.spearmanr
  role: Computes the Spearman rank correlations between sensitivity and specificity
    and between error score and alpha diversity deviation across pairings
- name: matplotlib
  role: Renders the genus- and species-level scatterplots with per-pairing centroid
    diamonds
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:01f948927b94956374faaec73845ae14737adbee9e9eb205beba5e02d47e6444
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  - matplotlib
  - scipy.stats.spearmanr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# taxonomic-profiler-benchmark-visualization

## Summary

Visualizes and quantifies how benchmark metrics co-vary across 16S profiler/reference-library pairings (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes/SILVA/RefSeq/Kraken Standard), using Spearman rank correlations of species-level sensitivity vs specificity and error score vs alpha diversity deviation, with scatterplots marking per-pairing centroids.

## When to use

Use when you have a per-pairing benchmark metrics table (sensitivity, specificity, error score, alpha diversity deviation for each profiler-library combination) and need to determine which metrics are correlated and which pairings occupy the high-sensitivity/high-specificity quadrant.

## When NOT to use

- Input is raw sequencing data or taxonomy calls rather than an already-computed benchmark metrics table — profiling must be run first (e.g., with PathoScope 2 or Kraken 2) before correlations can be computed
- The analysis goal is per-tool accuracy ranking alone (e.g., which library is most accurate) without examining cross-metric relationships — use a direct benchmark comparison instead
- The data are single-sample taxonomic profiles rather than aggregated mock-community benchmark metrics across the 136-sample mock community set

## Inputs

- Per-pairing benchmark metrics table with genus- and species-level sensitivity, specificity, error score, and alpha diversity deviation for each profiler-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq, Kraken Standard libraries)
- Mapping of each metric row to its profiler-library pairing (for centroid labeling)

## Outputs

- correlation_results.csv containing the two Spearman rank coefficients (species sensitivity vs specificity; error score vs alpha diversity deviation)
- holistic_trend_scatterplots.png with genus- and species-level scatterplots and per-pairing centroid diamonds

## How to apply

Load the benchmark metrics table containing genus- and species-level sensitivity, specificity, error score, and alpha diversity deviation for each of the profiler-library pairings (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 crossed with Greengenes, SILVA, RefSeq, and the Kraken Standard library). Compute Spearman rank correlations between species-level sensitivity and species-level specificity, and between error score and alpha diversity deviation, using scipy.stats.spearmanr. Generate matplotlib scatterplots at both genus and species levels, plotting sensitivity vs specificity and error vs alpha diversity deviation, and overlay a centroid diamond at the mean (x, y) coordinate for each profiler-library pairing so dominant profilers (PathoScope 2 with any library and Kraken 2 with its Standard library) are identifiable in the upper right quadrant. Save the two Spearman coefficients to correlation_results.csv and the figure to holistic_trend_scatterplots.png, validating that the coefficients reproduce the reported values (r = 0.85 for sensitivity-specificity, r = -0.57 for error vs alpha diversity deviation) within tolerance.

## Related tools

- **DADA2** (Benchmark profiler whose genus/species sensitivity, specificity, error score, and alpha diversity deviation metrics feed the correlation analysis and scatterplots)
- **QIIME 2** (Benchmark profiler (DADA2 plugin) contributing per-pairing metric rows to the metrics table being visualized)
- **Mothur** (Benchmark profiler contributing per-pairing metric rows to the metrics table being visualized)
- **PathoScope 2** (Benchmark profiler that dominates the high-sensitivity/high-specificity upper right quadrant; its per-pairing metrics are plotted and correlated) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Benchmark profiler (with Standard library) dominating the high-sensitivity/high-specificity quadrant; its per-pairing metrics are plotted and correlated) — github.com/aubreyodom/16SBenchmarking
- **scipy.stats.spearmanr** (Computes the Spearman rank correlations between sensitivity and specificity and between error score and alpha diversity deviation across pairings)
- **matplotlib** (Renders the genus- and species-level scatterplots with per-pairing centroid diamonds)

## Examples

```
from scipy.stats import spearmanr; r, p = spearmanr(df['species_sensitivity'], df['species_specificity'])
```

## Evaluation signals

- Spearman coefficient for species-level sensitivity vs specificity reproduces r = 0.85 within tolerance
- Spearman coefficient for error score vs alpha diversity deviation reproduces r = -0.57 within tolerance
- Both scatterplots (genus and species level) render with a centroid diamond at the mean (x, y) coordinate for every profiler-library pairing
- correlation_results.csv contains exactly the two reported coefficients; holistic_trend_scatterplots.png exists and is a valid image
- PathoScope (all reference libraries) and Kraken 2 (Standard library) points fall in the high-sensitivity/high-specificity upper right quadrant of the species-level plot

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so no analysis pipeline could perfectly measure a mock community
- Correlations are computed across a small number of profiler-library pairings, so the Spearman coefficients are sensitive to individual pairings
- Reference library quality affects the metrics: contamination in newer RefSeq versions and Greengenes' lower accuracy relative to SILVA and the Kraken 2 Standard library shape the plotted relationships

## Evidence

- [abstract] Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes: "Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes"
- [discussion] PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [readme] This respository contains links to relevant files for the Faits and Odom-Mabey et al. 16S pipeline benchmarking paper.: "This respository contains links to relevant files for the Faits and Odom-Mabey et al. 16S pipeline benchmarking paper."
