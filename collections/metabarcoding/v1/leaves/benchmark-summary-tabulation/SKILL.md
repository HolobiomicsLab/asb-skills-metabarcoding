---
name: benchmark-summary-tabulation
description: Use when per-sample taxon detection sensitivity and read assignment specificity
  values for multiple profiler/reference-library pairings on mock community data must
  be aggregated into paired genus- and species-level summary tables.
when_to_use_negative:
- Input is raw sequencing reads or unclassified per-sample outputs — classification
  must already have been run and per-taxon counts matrices produced first
- Data are real environmental/community microbiome samples without a mock ground-truth
  composition, since sensitivity and specificity here are defined against expected
  mock taxa at >=0.1% relative abundance
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: PathoScope 2
  role: Profiler whose per-pairing taxon counts (best performer, genus sensitivity
    mean 0.88 with RefSeq2018) feed the summary tabulation
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Profiler whose taxon counts matrices are collapsed to genus/species level
    and scored alongside PathoScope 2
  repo: github.com/aubreyodom/16SBenchmarking
- name: DADA2
  role: Profiler (QIIME 2 DADA2 plugin) whose outputs are scored in the same sensitivity/specificity
    framework
- name: QIIME 2
  role: Profiler whose taxon counts matrices are tabulated against ground truth
- name: Mothur
  role: Profiler whose taxon counts matrices are tabulated against ground truth
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:50e2ea27cc1c23a5dbeb492755b19ea71a912f31f200a94f7bd4c3bbd9bf027b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# benchmark-summary-tabulation

## Summary

Aggregates per-sample profiler performance (taxon detection sensitivity and read assignment specificity) into genus- and species-level means and standard deviations for each profiler–reference-library pairing, reproducing benchmark summary tables (Figure 2A–D, Supplementary Tables S2–S5).

## When to use

Use when per-sample taxon detection sensitivity and read assignment specificity values for multiple profiler/reference-library pairings on mock community data must be aggregated into paired genus- and species-level summary tables.

## When NOT to use

- Input is raw sequencing reads or unclassified per-sample outputs — classification must already have been run and per-taxon counts matrices produced first
- Data are real environmental/community microbiome samples without a mock ground-truth composition, since sensitivity and specificity here are defined against expected mock taxa at >=0.1% relative abundance

## Inputs

- Per-pairing taxon counts matrices from DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 classification runs
- Reference library taxonomy used per pairing (SILVA 138, Greengenes 13_8, Kraken Standard, RefSeq 2018/2020)
- Mock community ground-truth composition tables (136 mock samples)

## Outputs

- Summary table of genus-level taxon detection sensitivity means and SDs per profiler-library pairing (Figure 2A/2B, Supplementary Tables S2-S3)
- Summary table of species-level sensitivity and read assignment specificity means and SDs per pairing (Figure 2C/2D, Supplementary Tables S4-S5)

## How to apply

Load the per-pairing taxon counts matrices produced by DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 runs against the SILVA 138, Greengenes 13_8, Kraken Standard, and RefSeq (2018/2020) libraries, together with the mock community ground-truth composition tables for the 136 mock samples. Collapse each counts matrix to genus-level and species-level rows, tracing taxonomy back through the taxonomic hierarchy as needed for profilers reporting at finer or coarser ranks. For each sample and pairing, compute taxon detection sensitivity as the fraction of ground-truth taxa present at >=0.1% relative abundance that are detected by the profiler, and compute read assignment specificity as the fraction of reads assigned to taxa expected in the mock community ground truth. Aggregate per-sample values into means and standard deviations for each profiler-library pairing at each rank and tabulate the results. Validate by confirming the summary reproduces reported benchmarks (e.g., PathoScope+RefSeq2018 genus sensitivity mean = 0.88, SD = 0.14) within rounding tolerance.

## Related tools

- **PathoScope 2** (Profiler whose per-pairing taxon counts (best performer, genus sensitivity mean 0.88 with RefSeq2018) feed the summary tabulation) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Profiler whose taxon counts matrices are collapsed to genus/species level and scored alongside PathoScope 2) — github.com/aubreyodom/16SBenchmarking
- **DADA2** (Profiler (QIIME 2 DADA2 plugin) whose outputs are scored in the same sensitivity/specificity framework)
- **QIIME 2** (Profiler whose taxon counts matrices are tabulated against ground truth)
- **Mothur** (Profiler whose taxon counts matrices are tabulated against ground truth)

## Evaluation signals

- PathoScope paired with the RefSeq2018 library reproduces the reported genus-level detection sensitivity mean of 0.88 (SD = 0.14)
- Per-pairing means and SDs match Figure 2A-D and Supplementary Tables S2-S5 values within rounding tolerance
- Sensitivity denominators exclude ground-truth taxa below the 0.1% relative abundance threshold
- Read assignment specificity is bounded in [0,1] and computed only against taxa expected in the mock community ground truth
- All profiler-library pairings (5 profilers x 4 reference libraries) are represented in the tabulated summary at both genus and species levels

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors (pipetting variation during spike-in aliquoting, species-specific lysis difficulty, PCR amplification bias, reagent/environment contamination) are introduced at every experimental step, so no pipeline can perfectly measure a mock community
- Taxonomy collapsing requires tracing up the taxonomic hierarchy for profilers reporting at finer or coarser ranks, which can affect rank-level comparability across profilers
- Some reference libraries (e.g., newer RefSeq versions) require decontamination before use; the paper recommends older RefSeq libraries until contamination has been removed

## Evidence

- [full_text] PathoScope was the most sensitive method at detecting genera, peaking when paired with the RefSeq2018 reference library (mean = 0.88, SD = 0.14).: [evidence span withheld — anchor status: not_found]
- [full_text] compute sensitivity definition: [evidence span withheld — anchor status: not_found]
- [full_text] specificity definition: [evidence span withheld — anchor status: not_found]
- [full_text] collapse ranks: [evidence span withheld — anchor status: not_found]
- [discussion] mock ground truth caveat: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
- [discussion] refseq contamination caveat: "we recommend using older RefSeq libraries until newer versions have been processed to remove contamination"
- [readme] repo files: "This respository contains links to relevant files for the Faits and Odom-Mabey et al. 16S pipeline benchmarking paper."
