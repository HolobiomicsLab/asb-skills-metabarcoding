---
name: scatterplot-centroid-annotation
description: Use when a per-pairing benchmark metrics table (genus- and species-level
  sensitivity, specificity, error score, alpha diversity deviation for each profiler-library
  pairing) is available and you must assess trait correlation and quadrant dominance
  across pairings.
when_to_use_negative:
- Input is raw 16S amplicon FASTQ or per-read taxonomic assignments rather than an
  already-computed per-pairing benchmark metrics table — run the profiler benchmark
  first
- The goal is to actually classify reads with a profiler (DADA2, QIIME 2, Mothur,
  PathoScope 2, Kraken 2) rather than to summarize and visualize cross-pairing benchmark
  results
- You need per-sample or per-taxon detail; this skill operates at the level of profiler-library
  pairing summaries (centroids of mean metrics), not individual taxonomic calls
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3318
tools:
- name: scipy (scipy.stats.spearmanr)
  role: Computes the Spearman rank correlations between species-level sensitivity
    and specificity, and between error score and alpha diversity deviation, across
    profiler-library pairings
- name: matplotlib
  role: Renders the genus- and species-level scatterplots with per-pairing centroid
    diamond annotations and saves holistic_trend_scatterplots.png
- name: DADA2 / QIIME 2 / Mothur / PathoScope 2 / Kraken 2
  role: Upstream profilers whose per-pairing benchmark metrics (sensitivity, specificity,
    error score, alpha diversity deviation) populate the scatterplots and correlations
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:ddb0b214a06b254198bd82c58a8cc2867fdaf5677b54966e9ee8e2fe6e9667e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aubreyodom/16SBenchmarking
  tools:
  - DADA2
  - DADA2 / QIIME 2 / Mothur / PathoScope 2 / Kraken 2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  - matplotlib
  - scipy (scipy.stats.spearmanr)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3318
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# scatterplot-centroid-annotation

## Summary

A benchmark-summarization skill that computes Spearman rank correlations across profiler–reference-library pairings (e.g., DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes/SILVA/RefSeq-Kraken Standard) and renders sensitivity-vs-specificity and error-vs-alpha-diversity-deviation scatterplots with a centroid diamond at each pairing's mean (x, y) coordinate.

## When to use

Use when a per-pairing benchmark metrics table (genus- and species-level sensitivity, specificity, error score, alpha diversity deviation for each profiler-library pairing) is available and you must assess trait correlation and quadrant dominance across pairings.

## When NOT to use

- Input is raw 16S amplicon FASTQ or per-read taxonomic assignments rather than an already-computed per-pairing benchmark metrics table — run the profiler benchmark first
- The goal is to actually classify reads with a profiler (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) rather than to summarize and visualize cross-pairing benchmark results
- You need per-sample or per-taxon detail; this skill operates at the level of profiler-library pairing summaries (centroids of mean metrics), not individual taxonomic calls

## Inputs

- Per-pairing benchmark metrics table with genus-level and species-level sensitivity, specificity, error score, and alpha diversity deviation for each profiler-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq/Kraken Standard)

## Outputs

- correlation_results.csv containing the two Spearman rank coefficients (sensitivity-specificity r = 0.85; error vs alpha diversity deviation r = -0.57)
- holistic_trend_scatterplots.png with genus- and species-level scatterplots (sensitivity vs specificity and error vs alpha diversity deviation) and per-pairing centroid diamonds

## How to apply

Load the per-pairing benchmark metrics table covering the 16S profiler-library pairings (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq/Kraken Standard libraries). Compute Spearman rank correlations with scipy.stats.spearmanr, first between species-level sensitivity and species-level specificity (expected r = 0.85), then between error score and alpha diversity deviation (expected r = -0.57). Using matplotlib, generate scatterplots at both genus and species levels: one of sensitivity vs specificity and one of error vs alpha diversity deviation, overlaying a centroid diamond at the mean (x, y) coordinate for each profiler-library pairing so that upper-right (high-sensitivity/high-specificity) dominance is visually identifiable. Write the two Spearman coefficients to correlation_results.csv and the figure to holistic_trend_scatterplots.png. The rationale is that sensitivity and specificity are correlated traits at the species level, and the centroid diamonds expose which pairings (PathoScope regardless of library, Kraken 2 with its Standard library) occupy the high-quality quadrant.

## Related tools

- **scipy (scipy.stats.spearmanr)** (Computes the Spearman rank correlations between species-level sensitivity and specificity, and between error score and alpha diversity deviation, across profiler-library pairings)
- **matplotlib** (Renders the genus- and species-level scatterplots with per-pairing centroid diamond annotations and saves holistic_trend_scatterplots.png)
- **DADA2 / QIIME 2 / Mothur / PathoScope 2 / Kraken 2** (Upstream profilers whose per-pairing benchmark metrics (sensitivity, specificity, error score, alpha diversity deviation) populate the scatterplots and correlations) — github.com/aubreyodom/16SBenchmarking

## Examples

```
from scipy.stats import spearmanr; rho, _ = spearmanr(df.species_sensitivity, df.species_specificity)
```

## Evaluation signals

- Spearman coefficient for species-level sensitivity vs specificity reproduces the reported r = 0.85 within tolerance
- Spearman coefficient for error score vs alpha diversity deviation reproduces the reported r = -0.57 within tolerance
- correlation_results.csv exists and contains both coefficients in a parseable schema
- holistic_trend_scatterplots.png renders both panels (genus and species level) with a visible centroid diamond at the mean (x, y) coordinate for each profiler-library pairing
- PathoScope (all libraries) and Kraken 2 (Standard library) points fall in the upper right high-sensitivity/high-specificity quadrant of the species-level plot

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so the metrics being plotted are themselves imperfect references
- Contamination in reference libraries (e.g., RefSeq) demands additional filtering and correction after retrieval, which can shift pairing metrics; the article recommends using older RefSeq libraries until newer versions are cleaned
- Findings are grounded in 136 mock community samples and a fixed set of five profilers and four library families; correlations may not transfer to other pairings or real (non-mock) communities

## Evidence

- [abstract] PathoScope 2 and Kraken 2, both tools designed for whole-genome metagenomics, outperformed DADA2, QIIME 2 using the DADA2 plugin, and Mothur: "PathoScope 2 and Kraken 2, both tools designed for whole-genome metagenomics, outperformed DADA2, QIIME 2 using the DADA2 plugin, and Mothur"
- [discussion] PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [abstract] Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes: "Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes"
- [readme] PathoScope SILVA indices were custom-configured: "PathoScope SILVA indices were custom-configured"
