---
name: compositional-data-preprocessing
description: 'Use when a raw OTU/taxon-by-sample abundance table plus metadata must
  be cleaned before ANCOM-BC or normalization-comparison workflows: sparse taxa, low-library-size
  samples, and per-group structural zeros need handling.'
when_to_use_negative:
- group.var is continuous or ordinal — ANCOM-BC v1.0 only supports a discrete group.var
- The goal is DA testing for taxa that are structural zeros across all samples — ANCOM-BC/ANCOM-BC2
  are not designed to detect differences for such taxa (handled by the presence/absence
  test in the packaged version)
- Sample sizes are very small (e.g. n = 5 per group) and FDR control is critical
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: Provides feature_table_pre_process for filtering sparse taxa, low-library
    samples, and structural-zero detection prior to bias-corrected DA analysis
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Runtime environment for the ANCOM-BC scripts
- name: dplyr
  role: Required library dependency for the ANCOM-BC R code
- name: nloptr
  role: Required library dependency for the ANCOM-BC R code
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:59081263660be231f867ca971ce354a5184d4588680f92e006f81b2551608879
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM-BC
  - DESeq2
  - R
  - dplyr
  - edgeR
  - metagenomeSeq
  - nloptr
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# compositional-data-preprocessing

## Summary

Preprocess an absolute-abundance feature table and metadata with ANCOM-BC's feature_table_pre_process before differential abundance analysis, filtering sparse taxa and low-library samples and classifying structural zeros per group via an asymptotic lower bound.

## When to use

Use when a raw OTU/taxon-by-sample abundance table plus metadata must be cleaned before ANCOM-BC or normalization-comparison workflows: sparse taxa, low-library-size samples, and per-group structural zeros need handling.

## When NOT to use

- group.var is continuous or ordinal — ANCOM-BC v1.0 only supports a discrete group.var
- The goal is DA testing for taxa that are structural zeros across all samples — ANCOM-BC/ANCOM-BC2 are not designed to detect differences for such taxa (handled by the presence/absence test in the packaged version)
- Sample sizes are very small (e.g. n = 5 per group) and FDR control is critical

## Inputs

- Absolute abundance OTU/taxon table (data frame or matrix; taxa in rows, samples in columns)
- Metadata data frame containing the sample ID column (sample.var) and a discrete grouping variable (group.var)
- Parameter values: zero.cut (fraction between 0 and 1, e.g. 0.90), lib.cut (e.g. 1000), neg.lb (logical)

## Outputs

- feature.table: pre-processed OTU table data frame
- library.size: numeric vector of library sizes after pre-processing
- group.name: character vector of group.var levels
- group.ind: numeric vector assigning each sample to a group label
- structure.zeros: matrix of 0s and 1s, 1 indicating the taxon is a structural zero in the corresponding group

## How to apply

Call feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb) with the observed OTU table (taxa in rows, samples in columns) and a metadata frame; ANCOM-BC v1.0 supports only a discrete group.var. Set zero.cut=0.90 to drop taxa whose proportion of zeroes exceeds the threshold, lib.cut=1000 to drop samples with library size below the cutoff, and neg.lb=TRUE to flag structural zeros. With neg.lb=TRUE, for each taxon and group compute the non-zero proportion p_hat and declare a structural zero when p_hat equals 0 or when the asymptotic 95% lower bound p_hat - 1.96*sqrt(p_hat(1-p_hat)/n) falls at or below 0. The function returns the pre-processed feature.table, library.size, group.name, numeric group.ind, and a structure.zeros matrix of 0/1s (1 = structural zero in that group), which feed directly into ANCOM_BC via the grp.name, grp.ind, and struc.zero arguments.

## Related tools

- **ANCOM-BC** (Provides feature_table_pre_process for filtering sparse taxa, low-library samples, and structural-zero detection prior to bias-corrected DA analysis) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Runtime environment for the ANCOM-BC scripts)
- **dplyr** (Required library dependency for the ANCOM-BC R code)
- **nloptr** (Required library dependency for the ANCOM-BC R code)

## Examples

```
pre <- feature_table_pre_process(feature.table=otu_absolute, meta.data=meta(pseq), sample.var='sample', group.var='nationality', zero.cut=0.90, lib.cut=1000, neg.lb=TRUE)
```

## Evaluation signals

- structure.zeros is a matrix consisting only of 0s and 1s, with 1 marking a structural zero in the corresponding group (neg.lb rule: p_hat = 0 or p_hat - 1.96*sqrt(p_hat(1-p_hat)/n) <= 0)
- No taxon remaining in feature.table has proportion of zeroes greater than zero.cut
- No sample remaining has library size less than lib.cut
- group.ind values are integers 1..k matching the levels in group.name
- Downstream ANCOM_BC accepts the outputs directly as grp.name, grp.ind, and struc.zero without additional reshaping

## Limitations

- ANCOM-BC v1.0 only supports a discrete group.var
- Taxa that are structural zeros are significant only under the presence/absence test and are excluded from primary DA results
- ANCOM-BC may not perform well and FDR may not be controlled when sample sizes are very small, such as n = 5 per group
- Taxa with proportion of zeroes greater than zero.cut and samples with library size less than lib.cut are silently removed from the analysis

## Evidence

- [readme] TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound.: "TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound."
- [readme] Taxa with proportion of zeroes greater than zero.cut are not included in the analysis: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis."
- [readme] Samples with library size less than lib.cut are not included in the analysis: "Samples with library size less than lib.cut are not included in the analysis."
- [readme] structure.zeros: A matrix consists of 0 and 1s with 1 indicating the taxon is identified as a structural zero in the corresponding group.: "structure.zeros`` A matrix consists of 0 and 1s with 1 indicating the taxon is identified as a structural zero in the corresponding group."
- [methods] ANCOM-BC v1.0 only supports discrete group.var: "ANCOM-BC v1.0 only supports discrete group.var"
- [discussion] The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group.: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
