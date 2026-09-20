---
name: double-zero-exclusion-in-taxon-comparison
description: Use when computing accuracy-based error metrics (e.g., log NRMSE) between
  a profiler's taxonomic abundance table and a mock community's true composition,
  so that taxa absent in both are not treated as perfect matches.
when_to_use_negative:
- Input is already a filtered feature/error table with zero-count taxa removed — re-filtering
  is redundant
- Computing detection-based sensitivity/specificity at the sample level rather than
  accuracy-based error rates on per-taxon counts
- Comparing read-mapping quality or contamination screening rather than taxonomic
  abundance accuracy
tools:
- name: DADA2
  role: 16S amplicon profiler whose abundance table is compared against mock truth
    after double-zero exclusion
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: 16S amplicon profiler (DADA2 plugin) whose abundance table is compared against
    mock truth after double-zero exclusion
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: 16S amplicon profiler whose abundance table is compared against mock truth
    after double-zero exclusion
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: Metagenomic profiler whose abundance table is compared against mock truth
    after double-zero exclusion
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Metagenomic profiler whose abundance table is compared against mock truth
    after double-zero exclusion
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:b56bfad69dc1ead0aca4d79a0f2c6332d93c20526e2d8fcdc14104c7bf134247
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# double-zero-exclusion-in-taxon-comparison

## Summary

A data-preparation rule for accuracy-based comparison of measured vs. ground-truth mock community taxonomic abundance tables (from DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2): restrict each comparison to the union of expected and detected taxa and exclude double-zero pairs where both measured and true counts are zero before computing log NRMSE.

## When to use

Use when computing accuracy-based error metrics (e.g., log NRMSE) between a profiler's taxonomic abundance table and a mock community's true composition, so that taxa absent in both are not treated as perfect matches.

## When NOT to use

- Input is already a filtered feature/error table with zero-count taxa removed — re-filtering is redundant
- Computing detection-based sensitivity/specificity at the sample level rather than accuracy-based error rates on per-taxon counts
- Comparing read-mapping quality or contamination screening rather than taxonomic abundance accuracy

## Inputs

- Measured taxonomic abundance tables (read counts) per sample from each profiler/reference-library pair (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x SILVA 138, Greengenes 13_8, RefSeq 2018/2020, Kraken Standard)
- Ground-truth mock community composition tables (136 samples of publicly available synthetic mock communities)

## Outputs

- Filtered per-sample taxon count pairs (union of expected and detected taxa, double-zero pairs removed)
- Log NRMSE values per sample per pipeline/library pair at genus and species levels
- Aggregated mean log NRMSE table per pipeline/library pair (e.g., nrmse_figure2ef.csv)

## How to apply

Load the measured taxonomic abundance tables produced by each profiler/library pair (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 paired with SILVA 138, Greengenes 13_8, RefSeq 2018/2020, or the Kraken Standard library) along with the ground-truth mock community compositions from the 16SBenchmarking repository. Build the union of expected and detected taxa per sample and restrict every profiler-vs-truth comparison to that set. Drop double-zero pairs where both the measured and true read counts are zero, because counting them would inflate agreement between profiler and truth. Then compute the Normalized Root Mean Squared Error between measured and true read counts for every sample and each pipeline/library pair, apply a log transform to the NRMSE values, and aggregate into per-pair means at the genus and species levels. This per-taxon masking is applied identically across all pairs so error rates are comparable, as done for the Figure 2E-F benchmarks.

## Related tools

- **DADA2** (16S amplicon profiler whose abundance table is compared against mock truth after double-zero exclusion) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (16S amplicon profiler (DADA2 plugin) whose abundance table is compared against mock truth after double-zero exclusion) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (16S amplicon profiler whose abundance table is compared against mock truth after double-zero exclusion) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (Metagenomic profiler whose abundance table is compared against mock truth after double-zero exclusion) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Metagenomic profiler whose abundance table is compared against mock truth after double-zero exclusion) — github.com/aubreyodom/16SBenchmarking

## Examples

```
import pandas as pd; df = pd.read_csv('counts.csv'); filt = df[(df.true_count > 0) | (df.measured_count > 0)]
```

## Evaluation signals

- No remaining rows where both measured and true counts are zero after filtering
- Each comparison restricted to the union of expected and detected taxa for the given sample
- Aggregated mean log NRMSE reproduces Figure 2E-F benchmarks within numerical tolerance (Kraken 2+Standard species mean = 3.77; QIIME 2+Greengenes genus mean = 5.36)
- Output table contains one row per pipeline/library pair at each taxonomic level (genus and species)
- NRMSE values are log-transformed before aggregation

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step (pipetting variation, lysis bias, PCR amplification bias, reagent/environmental contamination), so true counts are not exact
- Some reference libraries (e.g., newer RefSeq versions) contain contamination that can distort measured taxa before comparison
- Excluding double-zero taxa means taxa missed by both profiler and library are not penalized, which can flatter poorly performing pairs

## Evidence

- [methods] Restrict each comparison to the union of expected and detected taxa, excluding double-zero pairs where both measured and true counts are zero.: [evidence span withheld — anchor status: not_found]
- [methods] Compute NRMSE then log transform: [evidence span withheld — anchor status: not_found]
- [abstract] Mock community ground truth scope: "We used 16S sequencing data from mock bacterial communities to evaluate the sensitivity and specificity"
- [discussion] Mock communities only approximate truth: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
