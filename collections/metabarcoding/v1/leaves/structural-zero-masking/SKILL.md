---
name: structural-zero-masking
description: Use when a taxa-by-sample absolute abundance table contains zeros that
  may be structural (taxon truly absent in a group) rather than sampling zeros, before
  computing group log differences or fitting the ANCOM-BC E-M mixture model.
when_to_use_negative:
- The goal is to test differential abundance of a taxon that is a structural zero
  in one group — ANCOM-BC is not designed to detect significant differences for such
  taxa, and they are only summarized via the presence/absence test
- Zeros are sampling zeros in taxa present (non-zero prevalence) in all groups — neg.lb
  will not flag them and they should remain in the E-M fit
- 'Input is raw unprocessed data: structural-zero classification requires the pre-processed
  table and asymptotic lower bounds from feature_table_pre_process, and taxa with
  zero proportion > zero.cut are already removed'
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ANCOM-BC (Code Archive)
  role: Provides feature_table_pre_process and ANCOM_BC, which classify structural
    zeros via neg.lb's asymptotic lower bound and accept the struc.zero matrix for
    masking
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOMBC R package
  role: Bioconductor implementation exposing structural-zero detection and the zero_ind
    presence/absence output in ancombc/ancombc2
  repo: https://github.com/FrederickHuangLin/ANCOMBC
- name: R
  role: Runtime environment for the ANCOM-BC scripts and library(dplyr)/library(nloptr)
    dependencies
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:d319b5f80200014b10f3ef0823656ed2bd5779c7a3c3851a943526cf6030a69c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM-BC
  - ANCOM-BC (Code Archive)
  - ANCOMBC R package
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# structural-zero-masking

## Summary

Identifies taxa that are genuinely absent (structural zeros) in a given experimental group using the asymptotic lower bound (neg.lb) computed by feature_table_pre_process, and masks those taxa–group entries so they are excluded from downstream ANCOM-BC log-scale mean-difference estimation and the E-M bias fit.

## When to use

Use when a taxa-by-sample absolute abundance table contains zeros that may be structural (taxon truly absent in a group) rather than sampling zeros, before computing group log differences or fitting the ANCOM-BC E-M mixture model.

## When NOT to use

- The goal is to test differential abundance of a taxon that is a structural zero in one group — ANCOM-BC is not designed to detect significant differences for such taxa, and they are only summarized via the presence/absence test
- Zeros are sampling zeros in taxa present (non-zero prevalence) in all groups — neg.lb will not flag them and they should remain in the E-M fit
- Input is raw unprocessed data: structural-zero classification requires the pre-processed table and asymptotic lower bounds from feature_table_pre_process, and taxa with zero proportion > zero.cut are already removed

## Inputs

- Pre-processed taxa-by-sample absolute abundance feature table (OTUs in rows, samples in columns)
- meta.data data frame with sample IDs and group.var levels
- zero.cut (taxa with proportion of zeroes greater than zero.cut are excluded), lib.cut (samples below library-size cutoff excluded), and neg.lb (logical) parameters
- grp.name (group levels) and grp.ind (numeric group assignment per sample) from feature_table_pre_process

## Outputs

- struc.zero matrix of 0s and 1s with 1 indicating the taxon is a structural zero in the corresponding group
- Masked Delta_i matrix (taxa-level log-scale group differences with structural-zero entries excluded)
- Separate presence/absence (zero_ind) summary for structural-zero taxa, excluded from the primary ANCOM_BC res table

## How to apply

Run feature_table_pre_process on the OTU table with meta.data, sample.var, group.var, zero.cut, lib.cut and neg.lb; when neg.lb=TRUE, a taxon is classified as a structural zero in a group using its asymptotic lower bound, producing the struc.zero matrix of 0s and 1s. Before computing taxa-level differences Delta_i on the natural log scale between groups, mask (set to NA/exclude) every Delta_i entry flagged as 1 in struc.zero so absent taxa do not contaminate the three-component Gaussian mixture fitted by the E-M algorithm (tol.EM = 1e-5, max.iterNum = 100). Structural-zero taxa are not tested for differential abundance in the primary mean.difference results; they are summarized separately (e.g. via zero_ind / presence-absence test). The matrix is also passed directly to ANCOM_BC as the struc.zero argument.

## Related tools

- **ANCOM-BC (Code Archive)** (Provides feature_table_pre_process and ANCOM_BC, which classify structural zeros via neg.lb's asymptotic lower bound and accept the struc.zero matrix for masking) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOMBC R package** (Bioconductor implementation exposing structural-zero detection and the zero_ind presence/absence output in ancombc/ancombc2) — https://github.com/FrederickHuangLin/ANCOMBC
- **R** (Runtime environment for the ANCOM-BC scripts and library(dplyr)/library(nloptr) dependencies)

## Examples

```
feature_table_pre_process(feature.table = otu_absolute, meta.data, sample.var = "sample", group.var = "nationality", zero.cut, lib.cut, neg.lb = TRUE)
```

## Evaluation signals

- struc.zero is a binary matrix (only 0s and 1s) with dimensions taxa x group levels and 1-entries only for taxa whose asymptotic lower bound crosses zero in that group
- Every Delta_i entry flagged 1 in struc.zero is masked (NA) before the three-component Gaussian mixture E-M fit; no structural-zero taxon appears in the primary mean.difference/W/p.val results
- The E-M algorithm converged within max.iterNum = 100 iterations at tol.EM = 1e-5, yielding finite bias.em and bias.wls estimates of the log sampling-fraction difference
- Structural-zero taxa are reported separately (zero_ind / presence-absence test), not in diff.abn
- d (estimated log sampling fractions) and mu (per-group log mean absolute abundances) returned by ANCOM_BC are consistent only over taxa retained after structural-zero masking

## Limitations

- The ANCOM-BC methodology may not perform well when sample sizes are very small, such as n = 5 per group
- ANCOM-BC v1.0 only supports a discrete group.var, so structural-zero classification is limited to discrete experimental groups
- Structural zeros are summarized separately rather than tested for differential abundance, so taxa truly absent in one group cannot receive a mean.difference p-value

## Evidence

- [readme] struc.zero matrix classification: "A matrix consists of 0 and 1s with 1 indicating the taxon is identified as a structural zero in the corresponding group."
- [readme] neg.lb lower-bound rule: "neg.lb`` Logical. TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound."
- [readme] structural zeros excluded from primary DA results: [evidence span withheld — anchor status: not_found]
- [methods] zero.cut pre-filter: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [discussion] structural zeros not tested for DA differences: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
