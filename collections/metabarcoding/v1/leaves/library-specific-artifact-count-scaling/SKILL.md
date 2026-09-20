---
name: library-specific-artifact-count-scaling
description: Use when a denoised or clustered mock-community feature table still contains
  unexpected mock ASVs ('miss' variants such as partial/miss ASVs) that the standard
  low-abundance filter did not remove, and you need an empirically derived per-library
  cutoff to zero them out.
when_to_use_negative:
- Input is raw reads or unfiltered output — this filter operates on an already-denoised/clustered
  ASV/OTU frequency table that has gone through the standard singleton-drop and ≥5,000-read
  filter first.
- There is no mock community in the dataset — the per-library subtraction integer
  is derived from observed 'miss' mock variant maxima and cannot be computed from
  diet samples alone.
- The goal is only the standard low-abundance filter; the extra per-library subtraction
  is a separate, more aggressive step for removing unexpected mock variants.
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: R
  role: Environment in which the per-library count subtraction and zeroing of the
    feature table are performed
- name: QIIME2R
  role: Used to load the prior task's mock-community ASV frequency table into R for
    filtering
- name: tidyverse
  role: Used for the per-library table manipulation and filtering steps in R
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:3b81efeaa58e2a09307bf6bd854b7d0a11dd1d7411c38a612229f1faebcc1c79
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2R
  - R
  - Tidyverse
  - tidyverse
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Library-specific artifact count scaling

## Summary

An extra filtering step for mock-community ASV/OTU feature tables that combines a standard filter (drop singletons across the dataset, retain samples with ≥5,000 filtered reads) with a per-library subtraction of the maximum count observed for unexpected ('miss') mock variants, zeroing any negative or zero results. It empirically removes low-abundance unexpected sequences (partial and miss ASVs) that survive denoising/clustering.

## When to use

Use when a denoised or clustered mock-community feature table still contains unexpected mock ASVs ('miss' variants such as partial/miss ASVs) that the standard low-abundance filter did not remove, and you need an empirically derived per-library cutoff to zero them out.

## When NOT to use

- Input is raw reads or unfiltered output — this filter operates on an already-denoised/clustered ASV/OTU frequency table that has gone through the standard singleton-drop and ≥5,000-read filter first.
- There is no mock community in the dataset — the per-library subtraction integer is derived from observed 'miss' mock variant maxima and cannot be computed from diet samples alone.
- The goal is only the standard low-abundance filter; the extra per-library subtraction is a separate, more aggressive step for removing unexpected mock variants.

## Inputs

- Denoised/clustering mock-community ASV or OTU frequency (feature) table from a prior task, with samples labeled by library
- Per-sample category assignments distinguishing expected mock ASVs from unexpected 'miss' mock ASVs (e.g. partial and miss ASVs)
- Library-to-mock-sample mapping used to compute per-library 'miss' count maxima

## Outputs

- Extra-filtered feature table (BIOM/TSV export) in which all unexpected 'miss' mock ASVs have zero abundance
- Set of retained samples, each meeting the ≥5,000 total filtered reads minimum after standard filtering

## How to apply

Load the denoised/clustering mock-community ASV/OTU frequency table from the prior task (e.g. via QIIME2R/pandas in R). First apply the paper's standard filter: drop any sequence variant observed in just one sample across the entire dataset and retain only samples with ≥5,000 total filtered reads. Next, on a per-library basis, determine the maximum count value observed for 'miss' (unexpected) mock sequence variants in that library's mock sample. Apply the extra filter by subtracting that single, fixed integer from each element of the feature table for that library, setting any negative or zero counts to zero. Rationale: denoising-vs-clustering error-rate differences are largely mitigated by removing low-abundance sequence variants, and mock communities provide an empirically derived threshold derived from the observed relative abundances of unexpected sequences. Finally export the filtered feature table (BIOM/TSV) as the canonical output.

## Related tools

- **R** (Environment in which the per-library count subtraction and zeroing of the feature table are performed)
- **QIIME2R** (Used to load the prior task's mock-community ASV frequency table into R for filtering)
- **tidyverse** (Used for the per-library table manipulation and filtering steps in R)

## Examples

```
ft_filtered <- ft - miss_max_per_library; ft_filtered[ft_filtered <= 0] <- 0
```

## Evaluation signals

- Every 'miss'-categorized mock ASV has exactly zero abundance in the exported filtered table.
- All retained samples meet the ≥5,000 total filtered reads minimum from the standard filter.
- Per library, every non-zero count in the filtered table is strictly greater than that library's maximum observed 'miss' variant count (the subtracted fixed integer).
- No negative values remain in the filtered table after zeroing negative and zero counts.
- The filtered table is a strict reduction relative to the standard-filter-only output (same retained variant set or smaller, with reduced counts).

## Limitations

- The subtracted integer is a per-library maximum from mock 'miss' variants; it can only zero out unexpected variants whose counts fall at or below that observed maximum in that library.
- Applying the extra filter discards legitimate low-count signal at or below the 'miss' maximum, so it trades sensitivity for artifact removal in the feature table.
- The approach is demonstrated for COI amplicon data (181 bp fragment) processed with denoising (DADA2/Deblur) or clustering (99%, 97%, 95% identity) outputs; its calibration is specific to the mock community used in this study.

## Evidence

- [methods] an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [methods] a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [intro] Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018): "Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018)"
- [abstract] denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
