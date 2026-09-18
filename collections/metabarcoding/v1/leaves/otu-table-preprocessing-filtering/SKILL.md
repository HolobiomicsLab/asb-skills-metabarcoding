---
name: otu-table-preprocessing-filtering
description: Use when an OTU/feature table (taxa in rows, samples in columns) must
  be cleaned and screened for structural zeros prior to ANCOM-BC differential abundance
  testing across discrete groups.
when_to_use_negative:
- The input is raw sequencing reads or already-normalized relative abundances rather
  than an absolute abundance count table
- The group variable of interest is continuous — ANCOM-BC v1.0 only supports discrete
  group.var
- Very small sample sizes (e.g. n = 5 per group), where ANCOM-BC does not control
  FDR well
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: Provides the feature_table_pre_process function that implements zero.cut/lib.cut
    filtering and structural-zero detection via asymptotic lower bounds
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Runtime environment; the preprocessing script sources dplyr and nloptr dependencies
- name: dplyr
  role: Required R library dependency for the ANCOM-BC preprocessing and analysis
    code
- name: nloptr
  role: Required R library dependency for the ANCOM-BC code archive scripts
merged_aliases:
- microbiome-otu-table-preprocessing
- otu-table-zero-filtering
merged_alias_records:
- alias: microbiome-otu-table-preprocessing
  slug: microbiome-otu-table-preprocessing
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
- alias: otu-table-zero-filtering
  slug: otu-table-zero-filtering
  cosine_score: 0.886
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:bdb4a92e81f9466769b751026d3c81e546c1101dc654699c42d3d341d1695016
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
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.36363636363636365
n_claims: 11
---

# otu-table-preprocessing-filtering

## Summary

Preprocess an absolute-abundance OTU table with ANCOM-BC's feature_table_pre_process before differential abundance testing: drop sparse taxa and low-library-size samples, and detect structural zeros per experimental group using asymptotic lower bounds.

## When to use

Use when an OTU/feature table (taxa in rows, samples in columns) must be cleaned and screened for structural zeros prior to ANCOM-BC differential abundance testing across discrete groups.

## When NOT to use

- The input is raw sequencing reads or already-normalized relative abundances rather than an absolute abundance count table
- The group variable of interest is continuous — ANCOM-BC v1.0 only supports discrete group.var
- Very small sample sizes (e.g. n = 5 per group), where ANCOM-BC does not control FDR well

## Inputs

- Absolute abundance OTU/feature table (data frame or matrix; taxa in rows, samples in columns)
- Sample metadata data frame containing sample IDs and a discrete group variable (e.g. nationality, bmi_group)
- Filter parameters: zero.cut = 0.90, lib.cut = 1000, neg.lb = TRUE; sample.var = 'sample'

## Outputs

- Pre-processed OTU feature table (data frame)
- Numeric vector of library sizes after pre-processing
- group.name: character vector of group.var levels
- group.ind: numeric vector assigning each sample to a group level
- structure.zeros: matrix of 0s and 1s (1 = structural zero in the corresponding group), e.g. saved as structure_zeros.rds

## How to apply

Call feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb) in R on the absolute abundance OTU table and metadata, with sample.var pointing to the column of sample IDs and group.var to a discrete grouping variable (e.g. 'nationality'). Apply zero.cut = 0.90 to exclude taxa whose proportion of zeroes exceeds 90%, lib.cut = 1000 to exclude samples with library size below 1000 reads, and neg.lb = TRUE so that a taxon is declared a structural zero in a group when the non-zero proportion p_hat equals 0 or when the asymptotic 95% lower bound p_hat - 1.96*sqrt(p_hat*(1-p_hat)/n) falls at or below 0. The function returns the pre-processed feature table, library sizes, group.name (levels of group.var), group.ind (numeric group labels per sample), and a structure.zeros matrix of 0s/1s (1 = structural zero in that group), which feed directly into ANCOM_BC via its grp.name, grp.ind, and struc.zero arguments. The preprocessing step must precede the ANCOM_BC main function because structural zeros should be flagged before bias-corrected log-scale modeling rather than imputed or tested naively.

## Related tools

- **ANCOM-BC** (Provides the feature_table_pre_process function that implements zero.cut/lib.cut filtering and structural-zero detection via asymptotic lower bounds) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Runtime environment; the preprocessing script sources dplyr and nloptr dependencies)
- **dplyr** (Required R library dependency for the ANCOM-BC preprocessing and analysis code)
- **nloptr** (Required R library dependency for the ANCOM-BC code archive scripts)

## Examples

```
feature.table.pre = feature_table_pre_process(feature.table = otu_absolute, meta.data = meta_data, sample.var = 'sample', group.var = 'nationality', zero.cut = 0.90, lib.cut = 1000, neg.lb = TRUE)
```

## Evaluation signals

- Returned structure.zeros is a matrix consisting only of 0s and 1s, with 1 indicating a structural zero in the corresponding group
- No retained taxon has a proportion of zeroes greater than zero.cut = 0.90 in the pre-processed table
- All samples in the pre-processed table have library size >= lib.cut = 1000
- group.ind values map 1:1 onto the levels listed in group.name and match the metadata group labels
- Structural-zero calls reproduce the neg.lb rule: p_hat == 0 or p_hat - 1.96*sqrt(p_hat*(1-p_hat)/n) <= 0

## Limitations

- ANCOM-BC may not perform well with very small sample sizes (n = 5 per group); FDR is not controlled in such cases
- ANCOM-BC v1.0 only supports discrete group.var
- Taxa with structural zeros cannot be tested for differential abundance by the main ANCOM_BC routine and must be handled separately

## Evidence

- [methods] zero.cut filter: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [methods] lib.cut filter: "Samples with library size less than lib.cut are not included in the analysis"
- [methods] neg.lb structural zeros: "TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound"
- [readme] structure.zeros output schema: "structure.zeros`` A matrix consists of 0 and 1s with 1 indicating the taxon is identified as a structural zero in the corresponding group."
- [methods] discrete group.var requirement: "ANCOM-BC v1.0 only supports discrete group.var"
