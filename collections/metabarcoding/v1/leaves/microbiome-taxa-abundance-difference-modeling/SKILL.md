---
name: microbiome-taxa-abundance-difference-modeling
description: Use when you have a pre-processed taxa-by-sample absolute abundance table
  plus group assignments and need to estimate/correct the sampling-fraction bias (delta)
  via the ANCOM-BC E-M mixture model before testing taxon differential abundance.
when_to_use_negative:
- 'Sample sizes are very small (e.g. n = 5 per group): ANCOM-BC may not perform well
  and FDR is not controlled in such cases'
- The taxon is a structural zero in a group — entries must be masked; ANCOM-BC is
  not designed to detect log-scale abundance differences for structural-zero taxa
- group.var is continuous — ANCOM-BC v1.0 only supports discrete group.var
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: ANCOM-BC
  role: Provides the ANCOM_BC function whose E-M step estimates the sampling-fraction
    bias term (bias.em) and the WLS variance approximation (bias.wls)
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Implementation language for the E-M algorithm and regression framework (R-package
    dependencies)
- name: dplyr
  role: Data manipulation library required by the ANCOM-BC code archive scripts
- name: nloptr
  role: Optimization library required by the ANCOM-BC code archive scripts
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:9476d52c7c93f90009b6e40435e89536ed8528e3b98ed8ea9ab81d614106749e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# microbiome-taxa-abundance-difference-modeling

## Summary

Reconstructs the E-M algorithm inside ANCOM-BC that estimates the sampling-fraction bias term delta — the mean difference of log (natural log) sampling fractions between groups — in a linear regression framework on the log scale, so that differential abundance of taxa in absolute abundance is bias-corrected.

## When to use

Use when you have a pre-processed taxa-by-sample absolute abundance table plus group assignments and need to estimate/correct the sampling-fraction bias (delta) via the ANCOM-BC E-M mixture model before testing taxon differential abundance.

## When NOT to use

- Sample sizes are very small (e.g. n = 5 per group): ANCOM-BC may not perform well and FDR is not controlled in such cases
- The taxon is a structural zero in a group — entries must be masked; ANCOM-BC is not designed to detect log-scale abundance differences for structural-zero taxa
- group.var is continuous — ANCOM-BC v1.0 only supports discrete group.var

## Inputs

- Pre-processed taxa-by-sample feature table (OTU/absolute abundance) from feature_table_pre_process
- grp.name: character vector of levels of group.var (e.g. nationality in dietswap)
- grp.ind: numeric vector assigning each sample to a group level
- struc.zero: matrix of 0/1 flagging structural zeros per taxon per group
- ANCOM_BC control parameters: adj.method, tol.EM = 1e-5, max.iterNum = 100, perNum, alpha

## Outputs

- bias.em: estimated mean difference of log (natural log) sampling fractions between groups via E-M
- bias.wls: mean difference of log sampling fractions via weighted least squares (variance approximation)
- d: numeric vector of estimated sampling fractions in log scale (natural log)
- mu: estimated log (natural log) mean absolute abundance per group
- res data frame: mean.difference, se, W, p.val, q.val, diff.abn per taxon
- em_bias_estimates.csv with d, mu, bias.em, bias.wls

## How to apply

Load the pre-processed feature table, group levels (grp.name), group assignments (grp.ind), and the structural-zero matrix from feature_table_pre_process (e.g. from the dietswap example data). Compute taxa-level differences Delta_i of absolute abundance on the natural log scale between groups, masking entries flagged as structural zeros in struc.zero. Fit a three-component Gaussian mixture model to Delta_i with an E-M algorithm using tol.EM = 1e-5 and a maximum of max.iterNum = 100 iterations, implemented with R, dplyr, and nloptr in ANCOM_BC. Extract the estimated bias delta as bias.em (mean difference of log sampling fractions between groups) and compute the weighted-least-squares approximation bias.wls for comparison; save estimated sampling fractions (d) and per-group log mean absolute abundances (mu). The log-scale regression with the sample-specific offset term is analogous to a log-ratio transformation to deal with compositionality, so do not apply additional compositional normalization.

## Related tools

- **ANCOM-BC** (Provides the ANCOM_BC function whose E-M step estimates the sampling-fraction bias term (bias.em) and the WLS variance approximation (bias.wls)) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Implementation language for the E-M algorithm and regression framework (R-package dependencies))
- **dplyr** (Data manipulation library required by the ANCOM-BC code archive scripts)
- **nloptr** (Optimization library required by the ANCOM-BC code archive scripts)

## Examples

```
ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method = "BH", tol.EM = 1e-5, max.iterNum = 100, perNum, alpha = 0.05)
```

## Evaluation signals

- Confirm the E-M converged within max.iterNum = 100 iterations at tol.EM = 1e-5
- Verify bias.em and bias.wls both represent the mean difference of log (natural log) sampling fractions between groups, with WLS providing the variance approximation
- Check structural-zero entries of struc.zero are masked out of the Delta_i computation before fitting the three-component Gaussian mixture
- Confirm outputs d (log sampling fractions) and mu (per-group log mean absolute abundance) are finite and consistent with the estimated bias

## Limitations

- ANCOM-BC may not perform well with very small sample sizes (n = 5 per group); FDR is not controlled in such cases
- Slightly elevated FDR when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var

## Evidence

- [intro] The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data"
- [discussion] it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions across samples: "it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions across samples"
- [readme] bias.em: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm.: "bias.em: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm."
- [readme] tol.EM: Numeric. The iteration convergence tolerance for E-M algorithm.: "tol.EM: Numeric. The iteration convergence tolerance for E-M algorithm."
- [discussion] The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group.: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
