---
name: spearman-rank-correlation-analysis
description: Use when a per-pairing benchmark metrics table exists (genus- and species-level
  sensitivity, specificity, error score, alpha diversity deviation for each profiler-library
  pairing) and the goal is to test whether quality traits are correlated across pipelines.
when_to_use_negative:
- Input is raw 16S amplicon sequencing reads (FASTQ) rather than an already-computed
  per-pairing benchmark metrics table
- The analysis requires per-sample abundance accuracy rather than cross-pipeline correlation
  of aggregate quality metrics
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: DADA2
  role: 16S profiler whose per-pairing benchmark metrics enter the correlation analysis
- name: QIIME 2
  role: 16S profiler (DADA2 plugin) whose per-pairing benchmark metrics enter the
    correlation analysis
- name: Mothur
  role: 16S profiler whose per-pairing benchmark metrics enter the correlation analysis
- name: PathoScope 2
  role: metagenomic profiler dominating the high-sensitivity/high-specificity quadrant;
    metrics feed the correlation analysis
- name: Kraken 2
  role: metagenomic profiler (with Standard library) dominating the high-sensitivity/high-specificity
    quadrant; metrics feed the correlation analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:6e2e0b38beb0568a3bc5bb828249403cdc3ba2a7050b81177b814d60c827007a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# spearman-rank-correlation-analysis

## Summary

Computes Spearman rank correlations between benchmark quality metrics (species-level sensitivity vs. specificity, and error score vs. alpha diversity deviation) across profiler/reference-library pairings for 16S amplicon data, and renders scatterplots with per-pairing centroid diamonds to reveal which pipeline-library pairs dominate the high-sensitivity/high-specificity quadrant.

## When to use

Use when a per-pairing benchmark metrics table exists (genus- and species-level sensitivity, specificity, error score, alpha diversity deviation for each profiler-library pairing) and the goal is to test whether quality traits are correlated across pipelines.

## When NOT to use

- Input is raw 16S amplicon sequencing reads (FASTQ) rather than an already-computed per-pairing benchmark metrics table
- The analysis requires per-sample abundance accuracy rather than cross-pipeline correlation of aggregate quality metrics

## Inputs

- Per-pairing benchmark metrics table with genus- and species-level sensitivity, specificity, error score, and alpha diversity deviation for each profiler-library pairing (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq/Kraken Standard libraries)

## Outputs

- correlation_results.csv containing the two Spearman rank coefficients (sensitivity-specificity and error vs. alpha diversity deviation)
- holistic_trend_scatterplots.png with genus- and species-level scatterplots and per-pairing centroid diamonds

## How to apply

Load the benchmark metrics table covering the five profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) crossed with reference libraries (Greengenes, SILVA, RefSeq, Kraken 2 Standard). Compute Spearman rank correlation between species-level sensitivity and species-level specificity across pairings with scipy.stats.spearmanr, and a second correlation between error score and alpha diversity deviation. Generate matplotlib scatterplots at genus and species levels plotting sensitivity vs. specificity and error vs. alpha diversity deviation, overlaying a centroid diamond at the mean (x, y) coordinate for each profiler-library pairing. Save the two Spearman coefficients to correlation_results.csv and the figure to holistic_trend_scatterplots.png. Validate by checking that the coefficients reproduce the reported values (r = 0.85 for sensitivity-specificity, r = -0.57 for error vs. alpha diversity deviation) within tolerance.

## Related tools

- **DADA2** (16S profiler whose per-pairing benchmark metrics enter the correlation analysis)
- **QIIME 2** (16S profiler (DADA2 plugin) whose per-pairing benchmark metrics enter the correlation analysis)
- **Mothur** (16S profiler whose per-pairing benchmark metrics enter the correlation analysis)
- **PathoScope 2** (metagenomic profiler dominating the high-sensitivity/high-specificity quadrant; metrics feed the correlation analysis)
- **Kraken 2** (metagenomic profiler (with Standard library) dominating the high-sensitivity/high-specificity quadrant; metrics feed the correlation analysis)

## Examples

```
from scipy.stats import spearmanr; r, p = spearmanr(df['sensitivity_species'], df['specificity_species'])
```

## Evaluation signals

- Spearman coefficient for species-level sensitivity vs. specificity reproduces r = 0.85 within tolerance
- Spearman coefficient for error score vs. alpha diversity deviation reproduces r = -0.57 within tolerance
- Both scatterplots render with a centroid diamond placed at the mean (x, y) coordinate for each profiler-library pairing
- correlation_results.csv contains exactly the two computed Spearman coefficients
- Number of points in the correlation equals the number of profiler-reference-library pairings evaluated

## Limitations

- Mock communities provide only a semblance of a ground truth; technical bias and errors are introduced at every experimental step, so correlation strength is bounded by benchmark quality
- Species relative abundances in mocks may be affected by pipetting variations, amplification bias during PCR, and variable lysis difficulty, causing underrepresentation or absence of some species
- Reference library contamination (e.g., E. coli 16S sequence in T. flagellates RefSeq scaffolds) can distort downstream metric values

## Evidence

- [other] Sensitivity and specificity are correlated traits at the species level (Spearman's r = 0.85), with PathoScope (regardless of reference library) and Kraken 2 (with its Standard library) dominating the upper right quadrant where both metrics are high.: [evidence span withheld — anchor status: not_found]
- [abstract] We used 16S sequencing data from mock bacterial communities to evaluate the sensitivity and specificity: "We used 16S sequencing data from mock bacterial communities to evaluate the sensitivity and specificity"
- [abstract] Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes: "Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes"
- [readme] PathoScope SILVA indices were custom-configured: "PathoScope SILVA indices were custom-configured, and can be downloaded [here.](https://rutgers.box.com/s/njq1qc9bzawzs3a6513uihxxwgxu9qc5)"
