---
name: genus-species-taxonomic-profiling-comparison
description: Use when 16S amplicon data from synthetic mock communities with known
  composition must be profiled by multiple profiler/library pairs and compared by
  accuracy-based genus- and species-level error rates (log NRMSE).
when_to_use_negative:
- The analysis goal is diversity-based (e.g., alpha/beta diversity) or sensitivity/specificity
  detection scoring rather than accuracy-based log NRMSE against mock community ground
  truth
- Input data are not 16S amplicon mock community samples with a known expected composition,
  so no true counts exist for NRMSE computation
- The task is building or configuring the reference indices themselves (e.g., Kraken
  2 Greengenes indices, PathoScope SILVA/RefSeq libraries) rather than benchmarking
  profiler accuracy with them
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: 16S amplicon processing pipeline evaluated as a profiler whose genus/species
    abundance calls are scored by log NRMSE
  repo: github.com/aubreyodom/16SBenchmarking
- name: QIIME 2
  role: Amplicon analysis pipeline (with DADA2 plugin) evaluated as a profiler; its
    Greengenes 13_8 pairing defines the genus-level benchmark value 5.36
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: Naive Bayes classifier-based 16S pipeline evaluated as a profiler in the pairwise
    NRMSE comparison
- name: PathoScope 2
  role: Whole-genome metagenomic read-mapping profiler evaluated against custom-configured
    SILVA, Greengenes, and RefSeq libraries
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: k-mer-based metagenomic classifier evaluated with SILVA, Greengenes, and Standard
    (RefSeq-comparable) indices; best species-level performance (mean log NRMSE =
    3.77 with Standard)
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:bcc24f511bc783a5c489ce59236b7f4c2b01a82c756bb395593f0e9812e904d9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# genus-species-taxonomic-profiling-comparison

## Summary

Benchmark 16S amplicon taxonomic profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) paired with reference libraries (SILVA 138, Greengenes 13_8, RefSeq 2018/2020, Kraken 2 Standard) by computing log-transformed NRMSE between measured and true mock-community read counts at genus and species levels.

## When to use

Use when 16S amplicon data from synthetic mock communities with known composition must be profiled by multiple profiler/library pairs and compared by accuracy-based genus- and species-level error rates (log NRMSE).

## When NOT to use

- The analysis goal is diversity-based (e.g., alpha/beta diversity) or sensitivity/specificity detection scoring rather than accuracy-based log NRMSE against mock community ground truth
- Input data are not 16S amplicon mock community samples with a known expected composition, so no true counts exist for NRMSE computation
- The task is building or configuring the reference indices themselves (e.g., Kraken 2 Greengenes indices, PathoScope SILVA/RefSeq libraries) rather than benchmarking profiler accuracy with them

## Inputs

- Measured taxonomic abundance tables per profiler/library pair (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x SILVA 138, Greengenes 13_8, RefSeq 2018/2020, Kraken 2 Standard)
- Ground-truth mock community composition tables (136 publicly available mock community samples with varied species richness and evenness)
- 16S amplicon sequencing FASTQ data from synthetic mock communities

## Outputs

- Per-sample log NRMSE values for each pipeline/library pair at genus and species levels
- Aggregated mean/SD log NRMSE table per pipeline/library pair and taxonomic level (nrmse_figure2ef.csv)
- Benchmark comparison against Figure 2E-F reference values

## How to apply

Load the measured taxonomic abundance tables produced by each profiler/library pair (DADA2, QIIME 2, Mothur, PathoScope 2, or Kraken 2, each paired with SILVA 138, Greengenes 13_8, RefSeq 2018/2020, or the Kraken 2 Standard library) alongside the ground-truth mock community compositions from the 16SBenchmarking repository. Restrict each comparison to the union of expected and detected taxa, excluding double-zero pairs where both measured and true counts are zero. Compute the Normalized Root Mean Squared Error between measured and true read counts per sample and per pair, then log-transform the nRMSE values. Aggregate the log nRMSE into means per pipeline/library pair at the genus and species levels, and validate against the paper's Figure 2E-F benchmarks (Kraken 2 + Standard species mean = 3.77, SD = 0.5; QIIME 2 + Greengenes genus mean = 5.36). Write the aggregated genus- and species-level table to nrmse_figure2ef.csv. The rationale is that accuracy-based measures on mock communities reveal which metagenomics-style tools (PathoScope 2, Kraken 2) and libraries (SILVA, RefSeq/Kraken Standard) outperform the conventional 16S tools and Greengenes.

## Related tools

- **DADA2** (16S amplicon processing pipeline evaluated as a profiler whose genus/species abundance calls are scored by log NRMSE) — github.com/aubreyodom/16SBenchmarking
- **QIIME 2** (Amplicon analysis pipeline (with DADA2 plugin) evaluated as a profiler; its Greengenes 13_8 pairing defines the genus-level benchmark value 5.36) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (Naive Bayes classifier-based 16S pipeline evaluated as a profiler in the pairwise NRMSE comparison)
- **PathoScope 2** (Whole-genome metagenomic read-mapping profiler evaluated against custom-configured SILVA, Greengenes, and RefSeq libraries) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (k-mer-based metagenomic classifier evaluated with SILVA, Greengenes, and Standard (RefSeq-comparable) indices; best species-level performance (mean log NRMSE = 3.77 with Standard)) — github.com/aubreyodom/16SBenchmarking

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- Kraken 2 + Standard species-level mean log NRMSE reproduces the reported value 3.77 (SD = 0.5) within numerical tolerance
- QIIME 2 + Greengenes genus-level mean log NRMSE reproduces the reported value 5.36 within numerical tolerance
- Double-zero pairs (taxa with zero measured AND zero true counts in a sample) are excluded before NRMSE computation
- nrmse_figure2ef.csv contains exactly one row per pipeline/library pair at each taxonomic level (genus and species)
- nRMSE values are log-transformed before per-pair aggregation, and the union of expected and detected taxa defines the comparison set

## Limitations

- Mock communities provide only a semblance of a 'ground truth'; technical bias and errors (pipetting variation when aliquoting spike-in DNA, species-specific lysis difficulty, PCR amplification bias, reagent/environmental contamination) are introduced at every experimental step
- RefSeq library versions should be checked for contamination; the paper recommends using older RefSeq libraries until newer versions have been processed to remove contamination (e.g., E. coli 16S rRNA sequence found in T. flagellates scaffolds)
- Greengenes 13_8 (August 2013) was the latest release benchmarked; Greengenes2 was released in 2022 amidst review and is not covered by this comparison

## Evidence

- [abstract] PathoScope 2 and Kraken 2, both tools designed for whole-genome metagenomics, outperformed DADA2, QIIME 2 using the DADA2 plugin, and Mothur: "PathoScope 2 and Kraken 2, both tools designed for whole-genome metagenomics, outperformed DADA2, QIIME 2 using the DADA2 plugin, and Mothur"
- [intro] Using several publicly available 16S sequencing datasets of synthetic mock communities, we specifically analyzed genus- and species-level performance: "Using several publicly available 16S sequencing datasets of synthetic mock communities, we specifically analyzed genus- and species-level performance across pairs of profilers and libraries"
- [discussion] our results present strong evidence that PathoScope and Kraken 2 outperform QIIME 2, Mothur, and DADA2: "our results present strong evidence that PathoScope and Kraken 2 outperform QIIME 2, Mothur, and DADA2, even when comparing reads to identical reference databases"
- [readme] Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper.: "Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper."
