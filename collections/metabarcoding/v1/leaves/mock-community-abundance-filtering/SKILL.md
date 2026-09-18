---
name: mock-community-abundance-filtering
description: Use when a denoised (DADA2/Deblur) or clustered (VSEARCH 99%/97%/95%)
  mock-community ASV/OTU feature table still contains unexpected 'miss' variants (e.g.,
  partial and miss ASVs) that must be removed before diversity analyses.
when_to_use_negative:
- Input is already a fully filtered feature table in which all unexpected mock ASVs
  are absent — re-applying the subtraction would remove real reads
- The dataset contains no mock community, so no empirically derived per-library maximum
  for 'miss' variants can be computed
- You are filtering the COI reference database rather than a feature table — the paper's
  database filters (remove gaps, keep >100 bp, require Family/Order taxonomy) apply
  there instead
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: R
  role: Language in which the standard and extra filtering steps are implemented
  repo: https://github.com/devonorourke/tidybug
- name: QIIME2R
  role: Imports the QIIME2 mock-community feature table (.qza) into R for filtering
- name: Tidyverse
  role: Data-frame manipulation for dropping single-sample ASVs, applying the >=5,000
    read threshold, and per-library subtraction
- name: VSEARCH
  role: Produces the 99%, 97%, and 95% identity clustered OTU tables that are inputs
    to this filtering workflow
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:c8dbc929c918627584fbac6948fcf70942151e940e6bda82650e069eedef2891
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/devonorourke/tidybug
  tools:
  - QIIME2R
  - R
  - Tidyverse
  - VSEARCH
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# mock-community-abundance-filtering

## Summary

Applies the tidybug paper's standard low-abundance filter (drop single-sample ASVs, keep ≥5,000-read samples) plus an extra per-library subtraction filter, using empirically derived maxima of unexpected 'miss' mock ASVs, to remove spurious sequence variants from mock-community COI feature tables.

## When to use

Use when a denoised (DADA2/Deblur) or clustered (VSEARCH 99%/97%/95%) mock-community ASV/OTU feature table still contains unexpected 'miss' variants (e.g., partial and miss ASVs) that must be removed before diversity analyses.

## When NOT to use

- Input is already a fully filtered feature table in which all unexpected mock ASVs are absent — re-applying the subtraction would remove real reads
- The dataset contains no mock community, so no empirically derived per-library maximum for 'miss' variants can be computed
- You are filtering the COI reference database rather than a feature table — the paper's database filters (remove gaps, keep >100 bp, require Family/Order taxonomy) apply there instead

## Inputs

- Denoised or clustered mock-community ASV/OTU feature table (QIIME2 .qza read via QIIME2R, BIOM, or TSV frequency table)
- Mock ASV classification labels distinguishing expected mock taxa from 'miss'/partial unexpected variants

## Outputs

- Filtered feature table (BIOM/TSV) in which every 'miss'-categorized mock ASV has zero abundance and all retained samples have >=5,000 total filtered reads

## How to apply

Load the denoised/clustering mock-community ASV frequency table (QIIME2 .qza feature table read via QIIME2R, or BIOM/TSV) with R/Tidyverse. First apply the standard filter: drop any sequence variant observed in just one sample across the entire dataset and retain only samples with >=5,000 total filtered reads. Then, on a per-library basis, determine the maximum count observed for 'miss' (unexpected) mock sequence variants in each library-specific mock sample, and apply the extra filter by subtracting that fixed integer from each element of the feature table for that library, zeroing negative or zero results. The rationale is that mock communities provide an empirically derived filtering strategy: the subtraction threshold reflects the observed relative abundances of unexpected sequences (partial and miss ASVs) so that all unexpected mock ASVs are zeroed in the canonical filtered table exported as BIOM/TSV.

## Related tools

- **R** (Language in which the standard and extra filtering steps are implemented) — https://github.com/devonorourke/tidybug
- **QIIME2R** (Imports the QIIME2 mock-community feature table (.qza) into R for filtering)
- **Tidyverse** (Data-frame manipulation for dropping single-sample ASVs, applying the >=5,000 read threshold, and per-library subtraction)
- **VSEARCH** (Produces the 99%, 97%, and 95% identity clustered OTU tables that are inputs to this filtering workflow) — https://github.com/torognes/vsearch

## Examples

```
Rscript -e 'library(qiime2R); ft <- read_qza("mock_feature_table.qza")$data; ft <- sapply(ft, function(x) pmax(x - max_miss_int, 0))'
```

## Evaluation signals

- Every 'miss'-categorized mock ASV has zero (or absent) abundance in the filtered output table
- All retained samples meet the >=5,000 total filtered reads minimum after the standard filter
- No sequence variant observed in exactly one sample across the entire dataset remains in the output
- No cell of the filtered feature table is negative after the per-library subtraction
- Comparing filtered vs. unfiltered tables shows a reduction in unexpected mock ASVs (partial and miss ASVs removed) while expected mock taxa are retained

## Limitations

- The extra filter requires a mock community with known expected composition; without labeled 'miss' variants no subtraction integer can be derived
- Denoising program and filtering choices interact — the number of mock ASVs observed in the dataset depends on both, so results are pipeline-dependent
- The subtraction is applied per library with a single fixed integer; unexpected variants above that maximum count in a library would survive the filter
- The paper's stated aim is not to present a single best pathway, but to illustrate how processes are affected by program or parameter choices

## Evidence

- [methods] a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads: "a standard filter required (a) dropping any sequence variant observed in just one sample across the entire dataset and (b) retaining only samples with ≥5,000 total filtered reads"
- [methods] an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table: "an extra filter incorporated the standard filters, and subtracted a single, fixed integer from each element of the feature table"
- [intro] Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018): "Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018)"
- [abstract] denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants: "denoising methods have lower error rates compared to traditional clustering methods, although these differences are largely mitigated by removing low‐abundance sequence variants"
- [readme] See [the docs folder of this repo](https://github.com/devonorourke/tidybug/tree/master/docs) for information on how each part of the project was performed: "See [the docs folder of this repo](https://github.com/devonorourke/tidybug/tree/master/docs) for information on how each part of the project was performed"
