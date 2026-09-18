---
name: asymptotic-lower-bound-inference
description: Use when preparing an OTU/feature table for ANCOM-BC differential abundance
  testing and you need to distinguish structural zeros (true absences in a group)
  from sampling zeros before fitting the bias-corrected model.
when_to_use_negative:
- The group variable is continuous — ANCOM-BC v1.0 only supports a discrete group.var
- You only need DA test statistics/p-values without structural-zero handling — the
  ANCOM_BC main function alone suffices
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC may not perform
  well and FDR is not controlled
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ANCOM-BC
  role: Provides feature_table_pre_process whose neg.lb parameter implements the asymptotic-lower-bound
    structural-zero rule
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Runtime environment; the preprocessing is executed as R code with dplyr and
    nloptr dependencies
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:1de17ae034b20764ce53cfce41b5db8fe3531398e17eb31c3f0abbd1d16bbd45
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# asymptotic-lower-bound-inference

## Summary

Classifies a taxon as a structural zero within an experimental group during ANCOM-BC preprocessing (feature_table_pre_process) by testing whether its estimated non-zero proportion's asymptotic 95% lower confidence bound falls at or below 0.

## When to use

Use when preparing an OTU/feature table for ANCOM-BC differential abundance testing and you need to distinguish structural zeros (true absences in a group) from sampling zeros before fitting the bias-corrected model.

## When NOT to use

- The group variable is continuous — ANCOM-BC v1.0 only supports a discrete group.var
- You only need DA test statistics/p-values without structural-zero handling — the ANCOM_BC main function alone suffices
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC may not perform well and FDR is not controlled

## Inputs

- Data frame or matrix of observed OTU table (taxa in rows, samples in columns), e.g. otu_absolute from a phyloseq object
- Data frame or matrix of metadata (meta_data) including sample IDs and a discrete group variable
- Parameters: sample.var, group.var, zero.cut (fraction between 0 and 1), lib.cut (numeric library-size cutoff), neg.lb (logical)

## Outputs

- structure.zeros matrix of 0s and 1s (1 = structural zero for that taxon in that group)
- Pre-processed feature table, library.size vector, group.name levels, and group.ind assignments (from feature_table_pre_process)

## How to apply

Run feature_table_pre_process on the absolute abundance table with sample.var, a discrete group.var, zero.cut (e.g., 0.90 to drop taxa with proportion of zeroes above the threshold), lib.cut (e.g., 1000 to drop low-library-size samples), and neg.lb=TRUE. For each taxon and group, compute the non-zero proportion p_hat and its asymptotic lower bound p_hat - 1.96*sqrt(p_hat(1-p_hat)/n); declare a structural zero when p_hat equals 0 or when this lower bound is non-positive. Extract the returned structure.zeros matrix, a matrix of 0s and 1s with 1 indicating the taxon is identified as a structural zero in the corresponding group, and pass it (struc.zero) to the ANCOM_BC main function so structural-zero taxa are summarized separately via the presence/absence test rather than the primary results.

## Related tools

- **ANCOM-BC** (Provides feature_table_pre_process whose neg.lb parameter implements the asymptotic-lower-bound structural-zero rule) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Runtime environment; the preprocessing is executed as R code with dplyr and nloptr dependencies)

## Examples

```
struc.zero <- feature_table_pre_process(feature.table = otu_absolute, meta.data = meta_data, sample.var = 'sample', group.var = 'nationality', zero.cut = 0.90, lib.cut = 1000, neg.lb = TRUE)$structure.zeros
```

## Evaluation signals

- Returned structure.zeros is a matrix containing only 0s and 1s, with 1 indicating a structural zero in the corresponding group
- A taxon is flagged 1 only when p_hat = 0 or p_hat - 1.96*sqrt(p_hat(1-p_hat)/n) <= 0 for that group
- Dimensions of structure.zeros match taxa rows and group.name levels after zero.cut/lib.cut filtering
- Taxa with proportion of zeroes greater than zero.cut are excluded before classification; samples with library size below lib.cut are removed first
- Structural-zero taxa appear only in the presence/absence (zero_ind) output, not as significant taxa in the primary ANCOM-BC results

## Limitations

- ANCOM-BC v1.0 only supports a discrete group.var
- ANCOM-BC may not perform well with very small sample sizes, such as n = 5 per group, and FDR is not controlled in such cases
- Taxa with structural zeros are considered significant only by the presence/absence test and are excluded from primary results

## Evidence

- [methods] TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound: "TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound"
- [readme] structure.zeros matrix definition: "A matrix consists of 0 and 1s with 1 indicating the taxon is identified as a structural zero in the corresponding group."
- [methods] zero.cut filter definition: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
