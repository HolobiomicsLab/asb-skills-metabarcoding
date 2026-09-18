---
name: relative-abundance-threshold-filtering
description: Use when computing taxon detection sensitivity for 16S amplicon mock
  communities (e.g., the 136-sample benchmark of DADA2, QIIME 2, Mothur, PathoScope
  2, Kraken 2 against SILVA/Greengenes/Kraken Standard/RefSeq libraries) and low-abundance
  ground-truth taxa must be excluded via a >=0.1%
when_to_use_negative:
- The input is raw 16S FASTQ reads rather than a taxon counts matrix paired with a
  mock community ground-truth composition table
- The goal is read assignment specificity, which is computed as the fraction of reads
  assigned to expected taxa and does not apply the 0.1% ground-truth abundance cutoff
- The dataset lacks a synthetic mock community with known composition, so no ground-truth
  abundance threshold can be defined
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: PathoScope 2
  role: Profiler whose genus-level detection counts are evaluated after threshold-filtering
    the mock community ground truth; most sensitive pairing was PathoScope + RefSeq2018
    (mean sensitivity 0.88, SD 0.14)
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Profiler whose genus/species-level detection counts are compared against the
    >=0.1% filtered ground-truth taxa set
  repo: github.com/aubreyodom/16SBenchmarking
- name: DADA2
  role: Profiler (also QIIME 2 plugin) whose detection results are scored against
    the threshold-filtered expected taxa
- name: QIIME 2
  role: Naive Bayes profiler whose genus/species-level outputs are collapsed and scored
    against filtered ground truth
- name: Mothur
  role: Naive Bayes profiler whose outputs are collapsed and scored against the threshold-filtered
    expected taxa
- name: MetaScope
  role: R package used to obtain up-to-date RefSeq indices for the profilers via download_refseq()
  repo: compbiomed.github.io/metascope-docs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:ae67326b39c638642d776e9f0e5becb3bb6a40e0e4dc574e1f6b8030a86da259
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aubreyodom/16SBenchmarking
  tools:
  - DADA2
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Relative-abundance-threshold-filtering

## Summary

Filters the mock-community ground-truth taxa to those present at >=0.1% relative abundance before computing per-sample taxon detection sensitivity, so sensitivity is judged against taxa that are reliably expected rather than against trace-level components distorted by experimental noise.

## When to use

Use when computing taxon detection sensitivity for 16S amplicon mock communities (e.g., the 136-sample benchmark of DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 against SILVA/Greengenes/Kraken Standard/RefSeq libraries) and low-abundance ground-truth taxa must be excluded via a >=0.1% relative-abundance cutoff.

## When NOT to use

- The input is raw 16S FASTQ reads rather than a taxon counts matrix paired with a mock community ground-truth composition table
- The goal is read assignment specificity, which is computed as the fraction of reads assigned to expected taxa and does not apply the 0.1% ground-truth abundance cutoff
- The dataset lacks a synthetic mock community with known composition, so no ground-truth abundance threshold can be defined

## Inputs

- Per-pairing taxon counts matrices from DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 classification runs
- Mock community ground-truth composition tables (136 mock samples)
- Taxonomic hierarchy mapping for collapsing counts to genus and species ranks

## Outputs

- Per-sample taxon detection sensitivity values restricted to ground-truth taxa at >=0.1% relative abundance, per profiler-library pairing
- Per-pairing means and standard deviations of sensitivity at genus and species levels
- Summary tables reproducing Figure 2A-D and Supplementary Tables S2-S5

## How to apply

Start from the per-pairing taxon counts matrices produced by DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 against the SILVA 138, Greengenes 13_8, Kraken Standard, and RefSeq (2018/2020) libraries, plus the mock community ground-truth composition tables. Collapse each counts matrix to genus-level and species-level rows, tracing taxonomy back through the taxonomic hierarchy for profilers that report at finer or coarser ranks. For each sample, apply the >=0.1% relative-abundance threshold to the ground-truth composition to define the set of expected taxa, then compute sensitivity as the fraction of those threshold-passing ground-truth taxa that the profiler detected. Read assignment specificity is computed separately as the fraction of reads assigned to expected taxa and does not use this cutoff. Aggregate per-sample values into means and standard deviations per profiler-library pairing at each rank, tabulating results that correspond to Figure 2A-D and Supplementary Tables S2-S5.

## Related tools

- **PathoScope 2** (Profiler whose genus-level detection counts are evaluated after threshold-filtering the mock community ground truth; most sensitive pairing was PathoScope + RefSeq2018 (mean sensitivity 0.88, SD 0.14)) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Profiler whose genus/species-level detection counts are compared against the >=0.1% filtered ground-truth taxa set) — github.com/aubreyodom/16SBenchmarking
- **DADA2** (Profiler (also QIIME 2 plugin) whose detection results are scored against the threshold-filtered expected taxa)
- **QIIME 2** (Naive Bayes profiler whose genus/species-level outputs are collapsed and scored against filtered ground truth)
- **Mothur** (Naive Bayes profiler whose outputs are collapsed and scored against the threshold-filtered expected taxa)
- **MetaScope** (R package used to obtain up-to-date RefSeq indices for the profilers via download_refseq()) — compbiomed.github.io/metascope-docs

## Examples

```
detected = sum(t in called for t, ab in ground_truth.items() if ab >= 0.001); sensitivity = detected / sum(ab >= 0.001 for ab in ground_truth.values())
```

## Evaluation signals

- The ground-truth taxa denominator and numerator only include taxa present at >=0.1% relative abundance in the mock composition table
- Sensitivity and specificity values are bounded in [0,1] and aggregated as means/SDs over the 136 mock samples per pairing and per rank
- The summary table reproduces the reported benchmarks, e.g., PathoScope+RefSeq2018 genus sensitivity mean equals 0.88 (SD = 0.14)
- Tabulated means/SDs match the Figure 2A-D and Supplementary Table S2-S5 values within rounding tolerance
- Genus- and species-level rows are correctly traced up/down the taxonomic hierarchy for profilers reporting at different ranks

## Limitations

- Mock communities provide only a semblance of ground truth: pipetting variation when aliquoting spike-in DNA, species-specific lysis difficulty, and PCR amplification bias can shift relative abundances, so a 0.1% cutoff may exclude taxa that are genuinely present or include distorted ones
- Profilers report at finer or coarser ranks than genus/species, so taxonomy must be traced through the hierarchy, which can propagate assignment errors
- Contamination in RefSeq libraries (e.g., T. flagellates scaffolds carrying E. coli 16S sequence) can inflate false detections; older RefSeq versions are recommended until newer ones are cleaned

## Evidence

- [other] Sensitivity threshold on ground truth: [evidence span withheld — anchor status: not_found]
- [abstract] Mock community dataset scope: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [intro] Benchmarking design: "Using several publicly available 16S sequencing datasets of synthetic mock communities, we specifically analyzed genus- and species-level performance across pairs of profilers and libraries"
- [discussion] Expected best performer under this metric: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [discussion] Mock communities approximate ground truth only: [evidence span withheld — anchor status: not_found]
