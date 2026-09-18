---
name: structural-zero-detection
description: Use when preparing an OTU/taxon abundance table (taxa in rows, samples
  in columns) with grouped metadata for ANCOM-BC, and taxa that are truly absent from
  some groups must be flagged as structural zeros prior to DA testing.
when_to_use_negative:
- Taxa flagged as structural zeros are handled by a presence/absence test only; ANCOM-BC
  is not designed to detect abundance-based differences for taxa with consistent zero
  counts across all samples in a group
- 'Very small sample sizes (e.g., n = 5 per group): ANCOM-BC may not perform well
  and FDR is not controlled'
- group.var is continuous — ANCOM-BC v1.0 only supports a discrete group.var
tools:
- name: R
  role: Language in which feature_table_pre_process is implemented; requires dplyr
    and nloptr libraries
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: Required R dependency for running the ANCOM-BC preprocessing code
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: Required R dependency for running the ANCOM-BC preprocessing code
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOMBC
  role: Packaged version of ANCOM-BC whose ancombc function reports structural zeros
    via the zero_ind output
  repo: github.com/FrederickHuangLin/ANCOMBC
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:a707e381e95a625ce07d2898a9a9ccfc096673c22567f434534205e7f7a60660
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - ANCOMBC
  - R
  - dplyr
  - nloptr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# structural-zero-detection

## Summary

Detects structural zeros (taxa absent by design in an experimental group) in a microbiome feature table using ANCOM-BC's feature_table_pre_process function with the neg.lb asymptotic-lower-bound rule, before differential abundance testing.

## When to use

Use when preparing an OTU/taxon abundance table (taxa in rows, samples in columns) with grouped metadata for ANCOM-BC, and taxa that are truly absent from some groups must be flagged as structural zeros prior to DA testing.

## When NOT to use

- Taxa flagged as structural zeros are handled by a presence/absence test only; ANCOM-BC is not designed to detect abundance-based differences for taxa with consistent zero counts across all samples in a group
- Very small sample sizes (e.g., n = 5 per group): ANCOM-BC may not perform well and FDR is not controlled
- group.var is continuous — ANCOM-BC v1.0 only supports a discrete group.var

## Inputs

- Feature table (data frame or matrix) of observed OTU/absolute abundances with taxa in rows and samples in columns
- Metadata data frame containing the sample ID column and a discrete group.var column (e.g., 'nationality')
- Parameters: sample.var (character), group.var (character), zero.cut (0.90), lib.cut (1000), neg.lb = TRUE

## Outputs

- structure.zeros matrix of 0s and 1s (1 = taxon is a structural zero in that group)
- Pre-processed feature.table (zero-cut taxa and low-library samples removed)
- library.size numeric vector after pre-processing
- group.name character vector of group.var levels and group.ind numeric group assignment per sample

## How to apply

Run feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb) in R with dependencies dplyr and nloptr loaded. Set zero.cut = 0.90 to drop taxa whose proportion of zeroes exceeds 90%, lib.cut = 1000 to drop samples with library size below 1000, and neg.lb = TRUE to enable structural-zero detection. For each taxon and group, compute the non-zero proportion p_hat; declare a structural zero when p_hat equals 0 or when the asymptotic 95% lower bound p_hat - 1.96*sqrt(p_hat(1-p_hat)/n) falls at or below 0. The rationale is that such taxa are absent by design rather than merely under-sampled, so they must be excluded from abundance-based inference (e.g., in ANCOM_BC, struc.zero is passed as a required argument).

## Related tools

- **R** (Language in which feature_table_pre_process is implemented; requires dplyr and nloptr libraries) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (Required R dependency for running the ANCOM-BC preprocessing code) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (Required R dependency for running the ANCOM-BC preprocessing code) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOMBC** (Packaged version of ANCOM-BC whose ancombc function reports structural zeros via the zero_ind output) — github.com/FrederickHuangLin/ANCOMBC

## Examples

```
pre <- feature_table_pre_process(feature.table = otu_absolute, meta.data = meta_data, sample.var = 'sample', group.var = 'nationality', zero.cut = 0.90, lib.cut = 1000, neg.lb = TRUE); structure_zeros <- pre$structure.zeros
```

## Evaluation signals

- Returned structure.zeros is a matrix consisting only of 0s and 1s, with rows as taxa and columns as groups
- Every taxon flagged 1 satisfies the neg.lb rule: p_hat = 0 or p_hat - 1.96*sqrt(p_hat(1-p_hat)/n) <= 0 in that group
- All taxa with proportion of zeroes > zero.cut (0.90) and all samples with library size < lib.cut (1000) are absent from the pre-processed table
- Passing the structure.zeros matrix as struc.zero to ANCOM_BC runs without errors, and structural-zero taxa are excluded from the abundance-based diff.abn results
- When using the packaged ancombc, structural-zero taxa appear in the zero_ind output rather than the primary results

## Limitations

- With very small sample sizes (e.g., n = 5 per group) ANCOM-BC may not perform well and FDR is not controlled
- Structural zeros are summarized separately and excluded from the primary ANCOM-BC/ancombc results; significance for such taxa is only assessable via the presence/absence test
- ANCOM-BC v1.0 only supports discrete group.var

## Evidence

- [readme] neg.lb rule: "TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound."
- [readme] output schema: "A matrix consists of 0 and 1s with 1 indicating the taxon is identified as a structural zero in the corresponding group."
- [methods] zero-cut filter: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [discussion] small-sample caveat: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
