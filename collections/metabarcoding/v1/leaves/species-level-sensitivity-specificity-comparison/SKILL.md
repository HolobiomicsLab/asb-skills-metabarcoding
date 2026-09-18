---
name: species-level-sensitivity-specificity-comparison
description: Use when benchmarking taxonomic profilers on mock community 16S data
  and you need to know whether sensitivity and specificity trade off or co-vary, and
  which profiler-library pairings sit in the high-both quadrant.
when_to_use_negative:
- Input is raw reads or ASV/OTU tables rather than an already-computed per-pairing
  metrics table — the skill compares summary benchmark metrics, not raw classifications
- The goal is de novo taxonomic profiling of a real (non-mock) community rather than
  benchmarking profilers against mock-community ground truth
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3307
tools:
- name: DADA2
  role: One of the benchmarked 16S profilers whose per-pairing sensitivity/specificity
    metrics feed the correlation and scatterplot analysis
- name: QIIME 2
  role: Benchmarked profiler (DADA2 plugin) contributing sensitivity, specificity,
    error score, and alpha diversity deviation values per reference library
- name: Mothur
  role: Benchmarked profiler contributing per-pairing species-level metrics to the
    Spearman analysis
- name: PathoScope 2
  role: Benchmarked metagenomic profiler dominating the high-sensitivity/high-specificity
    quadrant regardless of reference library
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Benchmarked metagenomic profiler; the Kraken Standard-library pairing dominates
    the high-both quadrant alongside PathoScope
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:8a9a7d801de17add83c5ecb5ea354bc631127616e1346dcbd732eac58001912a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aubreyodom/16SBenchmarking
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# species-level-sensitivity-specificity-comparison

## Summary

Benchmark profiler/reference-library pairings (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq/Kraken Standard) on mock 16S communities by comparing species-level sensitivity and specificity, then quantify their co-movement with Spearman correlation and quadrant scatterplots to identify pipelines that dominate the high-sensitivity/high-specificity region.

## When to use

Use when benchmarking taxonomic profilers on mock community 16S data and you need to know whether sensitivity and specificity trade off or co-vary, and which profiler-library pairings sit in the high-both quadrant.

## When NOT to use

- Input is raw reads or ASV/OTU tables rather than an already-computed per-pairing metrics table — the skill compares summary benchmark metrics, not raw classifications
- The goal is de novo taxonomic profiling of a real (non-mock) community rather than benchmarking profilers against mock-community ground truth

## Inputs

- Per-pairing benchmark metrics table (genus- and species-level sensitivity, specificity, error score, alpha diversity deviation) for DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq/Kraken Standard libraries
- Publicly available 16S mock community sequencing datasets (136 samples with varied species richness and evenness)

## Outputs

- correlation_results.csv containing the two Spearman rank coefficients (sensitivity-specificity r = 0.85; error vs alpha diversity deviation r = -0.57)
- holistic_trend_scatterplots.png: genus- and species-level scatterplots with per-pairing centroid diamonds

## How to apply

Load the per-pairing benchmark metrics table with genus- and species-level sensitivity, specificity, error score, and alpha diversity deviation for each profiler-library pairing. Compute Spearman rank correlations across pairings with scipy.stats.spearmanr: species-level sensitivity vs species-level specificity, and error score vs alpha diversity deviation. Render matplotlib scatterplots at genus and species levels (sensitivity vs specificity, error vs alpha diversity deviation), overlaying a centroid diamond at the mean (x, y) coordinate per profiler-library pairing so dominant pipelines are visually identifiable in the upper-right quadrant. Save coefficients to correlation_results.csv and the figure to holistic_trend_scatterplots.png. Accept the run when the coefficients reproduce the reported values (r = 0.85 for sensitivity-specificity, r = -0.57 for error vs alpha diversity deviation) within tolerance.

## Related tools

- **DADA2** (One of the benchmarked 16S profilers whose per-pairing sensitivity/specificity metrics feed the correlation and scatterplot analysis)
- **QIIME 2** (Benchmarked profiler (DADA2 plugin) contributing sensitivity, specificity, error score, and alpha diversity deviation values per reference library)
- **Mothur** (Benchmarked profiler contributing per-pairing species-level metrics to the Spearman analysis)
- **PathoScope 2** (Benchmarked metagenomic profiler dominating the high-sensitivity/high-specificity quadrant regardless of reference library) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Benchmarked metagenomic profiler; the Kraken Standard-library pairing dominates the high-both quadrant alongside PathoScope) — github.com/aubreyodom/16SBenchmarking

## Examples

```
from scipy.stats import spearmanr; r_ss, _ = spearmanr(df.species_sensitivity, df.species_specificity); r_ea, _ = spearmanr(df.error_score, df.alpha_diversity_deviation)
```

## Evaluation signals

- Computed Spearman coefficient for species-level sensitivity vs specificity matches r = 0.85 within tolerance
- Computed Spearman coefficient for error score vs alpha diversity deviation matches r = -0.57 within tolerance
- Both scatterplots render with a centroid diamond plotted at the mean (x, y) coordinate for each profiler-library pairing
- correlation_results.csv contains exactly the two Spearman coefficients and holistic_trend_scatterplots.png is produced
- PathoScope pairings and Kraken 2 (Standard library) appear in the upper-right quadrant of the species-level scatterplot

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step (pipetting of spike-in DNA, lysis difficulty, PCR amplification bias, reagent/environment contamination), so no pipeline can be measured perfectly
- Correlations are computed across a small set of profiler-library pairings, so coefficients describe these benchmarked pairings rather than a universal relationship
- Comparison is specific to genus- and species-level performance on 16S amplicon mock communities and does not transfer to whole-genome shotgun settings without re-evaluation

## Evidence

- [other] Sensitivity and specificity are correlated traits at the species level (Spearman's r = 0.85), with PathoScope (regardless of reference library) and Kraken 2 (with its Standard library) dominating the upper right quadrant: [evidence span withheld — anchor status: not_found]
- [abstract] We evaluated the tools DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 in conjunction with reference libraries: "We evaluated the tools DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 in conjunction with reference libraries"
- [intro] Using several publicly available 16S sequencing datasets of synthetic mock communities, we specifically analyzed genus- and species-level performance across pairs of profilers and libraries: "Using several publicly available 16S sequencing datasets of synthetic mock communities, we specifically analyzed genus- and species-level performance across pairs of profilers and libraries"
- [abstract] publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
