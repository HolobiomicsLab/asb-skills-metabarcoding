---
name: alpha-diversity-deviation-analysis
description: Use when benchmark metrics for multiple profiler/reference-library pairings
  are available and you need to test whether quality traits (sensitivity, specificity,
  error score, alpha diversity deviation) are correlated across pairings.
when_to_use_negative:
- Input is raw sequencing reads or a single profiler's output rather than an already-computed
  per-pairing benchmark metrics table
- The analysis goal is per-sample taxonomic profiling rather than cross-pairing quality-metric
  correlation
- Only one profiler-library pairing is available, leaving too few points for a meaningful
  Spearman rank correlation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3305
tools:
- name: DADA2
  role: 16S amplicon profiler whose pairing-level benchmark metrics feed the correlation
    analysis
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: 16S amplicon profiler (DADA2 plugin) whose pairing-level benchmark metrics
    feed the correlation analysis
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: 16S amplicon profiler whose pairing-level benchmark metrics feed the correlation
    analysis
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: metagenomic profiler whose pairing-level benchmark metrics feed the correlation
    analysis
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: metagenomic profiler whose pairing-level benchmark metrics feed the correlation
    analysis
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:4f9df703ac2170c63aea5641b625fbc6d0579041cb3fcfc02d0d753d704f5bcd
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3305
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# alpha-diversity-deviation-analysis

## Summary

Computes Spearman rank correlations between quality metrics (species-level sensitivity vs. specificity, error score vs. alpha diversity deviation) across 16S profiler-library pairings (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes/SILVA/RefSeq/Kraken Standard) and renders paired scatterplots with per-pairing centroid diamonds, showing sensitivity and specificity are correlated traits (r = 0.85).

## When to use

Use when benchmark metrics for multiple profiler/reference-library pairings are available and you need to test whether quality traits (sensitivity, specificity, error score, alpha diversity deviation) are correlated across pairings.

## When NOT to use

- Input is raw sequencing reads or a single profiler's output rather than an already-computed per-pairing benchmark metrics table
- The analysis goal is per-sample taxonomic profiling rather than cross-pairing quality-metric correlation
- Only one profiler-library pairing is available, leaving too few points for a meaningful Spearman rank correlation

## Inputs

- Per-pairing benchmark metrics table with genus- and species-level sensitivity, specificity, error score, and alpha diversity deviation for each profiler-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq/Kraken Standard)
- Profiler-library pairing labels for centroid annotation

## Outputs

- correlation_results.csv containing the two Spearman rank coefficients (sensitivity-specificity, error vs. alpha diversity deviation)
- holistic_trend_scatterplots.png with genus- and species-level scatterplots and per-pairing centroid diamonds

## How to apply

Load the per-pairing benchmark metrics table containing genus- and species-level sensitivity, specificity, error score, and alpha diversity deviation for each profiler-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 crossed with Greengenes, SILVA, RefSeq, and Kraken 2 Standard libraries). Compute the Spearman rank correlation between species-level sensitivity and species-level specificity across pairings using scipy.stats.spearmanr, and repeat for error score versus alpha diversity deviation. Generate scatterplots at genus and species levels with matplotlib — plotting sensitivity vs. specificity and error vs. alpha diversity deviation — and overlay a centroid diamond at the mean (x, y) coordinate for each profiler-library pairing to identify which pairings dominate the high-sensitivity/high-specificity upper-right quadrant (PathoScope regardless of library, and Kraken 2 with its Standard library). Save the two Spearman coefficients to correlation_results.csv and the figure to holistic_trend_scatterplots.png. Accept the result only when coefficients reproduce the reported values (r = 0.85 sensitivity-specificity, r = -0.57 error vs. alpha diversity deviation) within tolerance and both scatterplots render with per-pairing centroid diamonds.

## Related tools

- **DADA2** (16S amplicon profiler whose pairing-level benchmark metrics feed the correlation analysis) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (16S amplicon profiler (DADA2 plugin) whose pairing-level benchmark metrics feed the correlation analysis) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (16S amplicon profiler whose pairing-level benchmark metrics feed the correlation analysis) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (metagenomic profiler whose pairing-level benchmark metrics feed the correlation analysis) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (metagenomic profiler whose pairing-level benchmark metrics feed the correlation analysis) — github.com/aubreyodom/16SBenchmarking

## Examples

```
from scipy.stats import spearmanr; r, p = spearmanr(df['species_sensitivity'], df['species_specificity'])
```

## Evaluation signals

- Spearman coefficient for species-level sensitivity vs. specificity reproduces r = 0.85 within tolerance
- Spearman coefficient for error score vs. alpha diversity deviation reproduces r = -0.57 within tolerance
- correlation_results.csv contains exactly two coefficient records
- holistic_trend_scatterplots.png renders both genus- and species-level panels, each with a centroid diamond at the mean (x, y) for every profiler-library pairing
- PathoScope (all libraries) and Kraken 2 (Standard library) points fall in the upper-right high-sensitivity/high-specificity quadrant

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step
- Mock species relative abundances may be affected by subtle pipetting variations when aliquoting spike-in DNA
- Contamination from reagents and local environment (air, gloves, pipette tip boxes) can further complicate results, affecting the benchmark metrics fed into this analysis

## Evidence

- [other] Sensitivity and specificity are correlated traits at the species level (Spearman's r = 0.85), with PathoScope (regardless of reference library) and Kraken 2 (with its Standard library) dominating the upper right quadrant: [evidence span withheld — anchor status: not_found]
- [other] Load the per-pairing benchmark metrics table containing genus- and species-level sensitivity, specificity, error score, and alpha diversity deviation for each profiler-library pairing: [evidence span withheld — anchor status: not_found]
- [abstract] We used 16S sequencing data from mock bacterial communities to evaluate the sensitivity and specificity: "We used 16S sequencing data from mock bacterial communities to evaluate the sensitivity and specificity"
- [discussion] PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [readme] PathoScope SILVA indices were custom-configured, and can be downloaded: "PathoScope SILVA indices were custom-configured, and can be downloaded"
