---
name: nrmse-log-transform-aggregation
description: Use when benchmarking 16S amplicon taxonomic profilers (DADA2, QIIME
  2, Mothur, PathoScope 2, Kraken 2) against mock communities with known composition
  and you need accuracy-based, log-scaled error rates at genus or species level.
when_to_use_negative:
- Input is already a diversity- or detection-based benchmark result (sensitivity/specificity)
  rather than count-level abundance data needing nRMSE computation
- Taxonomic abundance tables cannot be mapped to a shared ground truth (no mock community
  composition available), since nRMSE requires expected vs. measured counts for the
  union of expected and detected taxa
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: DADA2
  role: 16S amplicon pipeline whose abundance tables are benchmarked via log nRMSE
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: 16S amplicon pipeline (DADA2 plugin) whose abundance tables are benchmarked
    via log nRMSE
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: 16S amplicon pipeline whose abundance tables are benchmarked via log nRMSE
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: Metagenomic profiler whose abundance tables are benchmarked via log nRMSE
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Metagenomic classifier whose abundance tables are benchmarked via log nRMSE
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:2a89850230bed3d07e740801b4b5acd593109a76a35dd77f1b50178f6727a52b
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# nrmse-log-transform-aggregation

## Summary

Compute the Normalized Root Mean Squared Error (nRMSE) between measured and true mock-community read counts for each 16S profiler/reference-library pair, log-transform the values, and aggregate into genus- and species-level means (e.g., Kraken 2 + Standard species mean = 3.77; QIIME 2 + Greengenes genus mean = 5.36) to reproduce the Figure 2E-F accuracy benchmarks.

## When to use

Use when benchmarking 16S amplicon taxonomic profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) against mock communities with known composition and you need accuracy-based, log-scaled error rates at genus or species level.

## When NOT to use

- Input is already a diversity- or detection-based benchmark result (sensitivity/specificity) rather than count-level abundance data needing nRMSE computation
- Taxonomic abundance tables cannot be mapped to a shared ground truth (no mock community composition available), since nRMSE requires expected vs. measured counts for the union of expected and detected taxa

## Inputs

- Measured taxonomic abundance tables (read counts per taxon) from DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2, each paired with SILVA 138, Greengenes 13_8, RefSeq 2018/2020, or the Kraken 2 Standard library
- Ground-truth mock community compositions (136 publicly available mock community samples with varied species richness and evenness)
- Taxonomic hierarchy mapping for genus- and species-level aggregation

## Outputs

- Per-sample, per-pipeline/library-pair log nRMSE values at genus and species levels
- Aggregated mean (and SD) log nRMSE table per pipeline/library pair, written to nrmse_figure2ef.csv with one row per pair at each taxonomic level

## How to apply

Load the measured taxonomic abundance tables produced by each profiler/library pair (DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 paired with SILVA 138, Greengenes 13_8, RefSeq 2018/2020, or the Kraken Standard library) and the corresponding ground-truth mock community compositions from the 16SBenchmarking repository. Restrict each comparison to the union of expected and detected taxa, excluding double-zero pairs where both measured and true counts are zero. Compute nRMSE between measured and true read counts per sample per pipeline/library pair, then log-transform the nRMSE values. Aggregate log nRMSE into means per pipeline/library pair at genus and species levels, and validate the aggregated means against reported Figure 2E-F benchmarks (Kraken 2+Standard species mean 3.77, QIIME 2+Greengenes genus mean 5.36) within numerical tolerance.

## Related tools

- **DADA2** (16S amplicon pipeline whose abundance tables are benchmarked via log nRMSE) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (16S amplicon pipeline (DADA2 plugin) whose abundance tables are benchmarked via log nRMSE) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (16S amplicon pipeline whose abundance tables are benchmarked via log nRMSE) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (Metagenomic profiler whose abundance tables are benchmarked via log nRMSE) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Metagenomic classifier whose abundance tables are benchmarked via log nRMSE) — github.com/aubreyodom/16SBenchmarking

## Evaluation signals

- Computed Kraken 2 + Standard species-level mean log nRMSE equals 3.77 (SD = 0.5) and SILVA equals 3.95 (SD = 0.55) within numerical tolerance
- Computed QIIME 2 + Greengenes genus-level mean log nRMSE equals 5.36 within numerical tolerance
- Output file nrmse_figure2ef.csv contains one row per pipeline/library pair at each taxonomic level
- Double-zero taxon pairs (both measured and true counts zero) are excluded from every comparison
- Kraken 2 log NRMSE values at species level are lower than all other methods evaluated

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors (pipetting variation, lysis difficulty, PCR amplification bias, contamination) are introduced at every experimental step
- No analysis pipeline could theoretically exist to perfectly measure a mock community, so nRMSE values cannot reach zero
- Reference library quality affects results: RefSeq 2018/2020 indices required custom configuration and newer RefSeq versions should be processed to remove contamination before use

## Evidence

- [results] Kraken 2 had the lowest log NRMSE with its Standard database (mean = 3.77; SD = 0.5) and SILVA (mean = 3.95, SD = 0.55): [evidence span withheld — anchor status: not_found]
- [abstract] Benchmarked tools and libraries: "We evaluated the tools DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 in conjunction with reference libraries"
- [discussion] Mock community ground truth caveat: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
- [readme] Reference library indices sources: "The Kraken Standard indices was originally downloaded on 8/20/2020."
- [intro] Benchmarking objective: "we systematically benchmarked several current community profiling tools and reference libraries created for both metagenomic and 16S analysis."
