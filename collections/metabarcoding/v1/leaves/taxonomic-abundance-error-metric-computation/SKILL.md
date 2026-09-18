---
name: taxonomic-abundance-error-metric-computation
description: Use when benchmarking 16S taxonomic profilers (DADA2, QIIME 2, Mothur,
  PathoScope 2, Kraken 2) against mock community ground truth and you need accuracy-based
  error (log NRMSE) at genus or species level.
when_to_use_negative:
- The goal is sensitivity/specificity of taxon detection rather than accuracy of abundance
  estimates (error score)
- Comparison is against real (non-mock) communities with no ground-truth composition,
  since no true counts exist to compute NRMSE against
- The input is already an aggregated error table (e.g., a pre-computed nrmse_figure2ef.csv);
  only verify or re-aggregate instead of recomputing
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: DADA2
  role: 16S amplicon profiler whose measured abundance tables are one input to the
    NRMSE comparison
- name: QIIME 2
  role: 16S amplicon profiler (DADA2 plugin) whose measured abundance tables are one
    input to the NRMSE comparison
- name: Mothur
  role: 16S amplicon profiler whose measured abundance tables are one input to the
    NRMSE comparison
- name: PathoScope 2
  role: Metagenomic profiler whose measured abundance tables are one input to the
    NRMSE comparison
- name: Kraken 2
  role: Metagenomic classifier whose measured abundance tables (with Standard/SILVA/Greengenes
    libraries) are one input to the NRMSE comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:0fbfce16a23a9cfe336870e2597175ac33c0c2d36e008c8f9d772755d49b8087
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
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# taxonomic-abundance-error-metric-computation

## Summary

Computes log-transformed Normalized Root Mean Squared Error (log NRMSE) between profiler-measured and ground-truth mock community read counts to quantify accuracy-based error of 16S profiler/reference-library pairings at genus and species levels.

## When to use

Use when benchmarking 16S taxonomic profilers (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) against mock community ground truth and you need accuracy-based error (log NRMSE) at genus or species level.

## When NOT to use

- The goal is sensitivity/specificity of taxon detection rather than accuracy of abundance estimates (error score)
- Comparison is against real (non-mock) communities with no ground-truth composition, since no true counts exist to compute NRMSE against
- The input is already an aggregated error table (e.g., a pre-computed nrmse_figure2ef.csv); only verify or re-aggregate instead of recomputing

## Inputs

- Measured taxonomic abundance tables (read counts per taxon per sample) from each profiler/library pair: DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x SILVA 138, Greengenes 13_8, RefSeq 2018, RefSeq 2020, Kraken Standard
- Ground-truth mock community compositions for the 136 publicly available synthetic mock community samples

## Outputs

- nrmse_figure2ef.csv: one row per pipeline/library pair at each taxonomic level (genus and species) with aggregated mean log NRMSE values

## How to apply

Load the measured taxonomic abundance tables produced by each profiler/library pair (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2, each paired with SILVA 138, Greengenes 13_8, RefSeq 2018/2020, or the Kraken Standard library) together with the corresponding ground-truth mock community compositions from the 16SBenchmarking repository. Restrict each comparison to the union of expected and detected taxa, excluding double-zero pairs where both measured and true counts are zero. For every sample and each pipeline/library pair, compute the Normalized Root Mean Squared Error between measured and true read counts, then apply a log transform to the nRMSE values. Aggregate log nRMSE into per-pair means at genus level and species level, and verify against the reported Figure 2E-F benchmarks (Kraken 2 + Standard species mean = 3.77, SD = 0.5; QIIME 2 + Greengenes genus mean = 5.36) before writing the final table.

## Related tools

- **DADA2** (16S amplicon profiler whose measured abundance tables are one input to the NRMSE comparison)
- **QIIME 2** (16S amplicon profiler (DADA2 plugin) whose measured abundance tables are one input to the NRMSE comparison)
- **Mothur** (16S amplicon profiler whose measured abundance tables are one input to the NRMSE comparison)
- **PathoScope 2** (Metagenomic profiler whose measured abundance tables are one input to the NRMSE comparison)
- **Kraken 2** (Metagenomic classifier whose measured abundance tables (with Standard/SILVA/Greengenes libraries) are one input to the NRMSE comparison)

## Examples

```
import numpy as np; log_nrmse = np.log(np.sqrt(np.mean((measured - true)**2)) / np.mean(true))  # per pair, then aggregate means to nrmse_figure2ef.csv
```

## Evaluation signals

- Kraken 2 + Standard database reproduces species-level mean log NRMSE = 3.77 (SD = 0.5) within numerical tolerance
- QIIME 2 + Greengenes reproduces genus-level mean log NRMSE = 5.36 within numerical tolerance
- Computed means match all Figure 2E-F reported values for every pipeline/library pairing
- Output nrmse_figure2ef.csv contains exactly one row per pipeline/library pair at each taxonomic level
- Double-zero pairs (both measured and true counts zero) are excluded from every per-sample NRMSE computation

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so no analysis pipeline could theoretically exist to perfectly measure a mock community
- Mock species relative abundances may be affected by subtle pipetting variations when aliquoting spike-in DNA, and species vary in lysis difficulty, causing underrepresentation or absence in collected libraries
- Reference libraries retrieved from public repositories may require additional filtering and correction of contamination (e.g., RefSeq contamination identified via pairwise BLAST)

## Evidence

- [abstract] Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes: "Evaluations of reference libraries identified the SILVA and RefSeq/Kraken 2 Standard libraries as superior in accuracy compared to Greengenes"
- [discussion] PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur: "PathoScope is more sensitive and specific at taxon detection and has a lower error score than either DADA2, QIIME 2, or Mothur"
- [intro] Using several publicly available 16S sequencing datasets of synthetic mock communities, we analyzed genus- and species-level performance across pairs of profilers and libraries: "Using several publicly available 16S sequencing datasets of synthetic mock communities, we specifically analyzed genus- and species-level performance across pairs of profilers and libraries"
- [readme] Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper.: "Note that this library is comparable to the PathoScope RefSeq 2020 library used in the paper."
