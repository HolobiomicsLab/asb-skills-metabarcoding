---
name: gaussian-mixture-expectation-maximization-estimation
description: Use when performing differential abundance analysis of compositional
  microbiome data where differences in sampling fractions across samples introduce
  bias that must be estimated from the data itself.
when_to_use_negative:
- 'Sample sizes are very small (e.g., n = 5 per group): the article notes ANCOM-BC
  may not perform well and FDR is not controlled in such cases'
- 'The taxon is flagged as a structural zero in a group: structural-zero entries are
  masked out and differences there should not drive the mixture fit'
- 'The variable of interest is continuous rather than discrete: ANCOM-BC v1.0 only
  supports a discrete group.var'
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: ANCOM-BC
  role: Method whose bias-estimation step (offset term in the log-scale linear regression)
    is implemented by the E-M Gaussian mixture fit
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Implementation language for the E-M algorithm and the ANCOM-BC code archive
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: R-package dependency required for running the ANCOM-BC archive scripts
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: R-package dependency providing numerical optimization used by the archive
    implementation
  repo: github.com/FrederikHuangLin/ANCOM-BC-Code-Archive
- name: ANCOMBC
  role: Packaged R implementation exposing ancombc / ancombc2 with the same bias-correction
    machinery
  repo: FrederickHuangLin/ANCOMBC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:04e9f3da09ba20c8b9951cc6c6c837af32158455fab049b3ca25f825bfee6e67
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# gaussian-mixture-expectation-maximization-estimation

## Summary

Fit a three-component Gaussian mixture model to taxa-level log mean differences via an E-M algorithm to estimate the sampling-fraction bias term (bias.em) in ANCOM-BC, with a weighted-least-squares estimate (bias.wls) and variance approximation for comparison.

## When to use

Use when performing differential abundance analysis of compositional microbiome data where differences in sampling fractions across samples introduce bias that must be estimated from the data itself.

## When NOT to use

- Sample sizes are very small (e.g., n = 5 per group): the article notes ANCOM-BC may not perform well and FDR is not controlled in such cases
- The taxon is flagged as a structural zero in a group: structural-zero entries are masked out and differences there should not drive the mixture fit
- The variable of interest is continuous rather than discrete: ANCOM-BC v1.0 only supports a discrete group.var

## Inputs

- Pre-processed OTU/taxon-by-sample feature table (data frame or matrix, taxa in rows, samples in columns)
- grp.name: character vector of group levels
- grp.ind: numeric vector of group assignment per sample
- struc.zero: 0/1 matrix of structural zeros per taxon per group
- ANCOM_BC tuning parameters: tol.EM, max.iterNum, adj.method, perNum, alpha

## Outputs

- bias.em: numeric, estimated mean difference of log (natural log) sampling fractions between groups via E-M
- bias.wls: numeric, same quantity via weighted least squares with variance approximation
- d: numeric vector of estimated sampling fractions in natural log scale
- mu: numeric vector of estimated log (natural log) mean absolute abundance per group
- res data frame with mean.difference, se, W test statistic, p.val, q.val, diff.abn
- em_bias_estimates.csv containing d, mu, and bias.em/bias.wls

## How to apply

Load the pre-processed taxa-by-sample feature table, group levels (grp.name), group assignments (grp.ind), and the structural-zero matrix from feature_table_pre_process (e.g., on the dietswap dataset). Compute taxa-level differences Delta_i on the natural log scale between groups, masking entries flagged as structural zeros in struc.zero. Fit a three-component Gaussian mixture model to Delta_i with an E-M algorithm using tol.EM = 1e-5 and a maximum of max.iterNum = 100 iterations (nloptr supplies the optimization machinery in the R code archive). Extract bias.em, the estimated mean difference of log (natural log) sampling fractions between groups, and compute bias.wls via weighted least squares for comparison and variance approximation. Save the estimated sampling fractions (d), per-group log mean absolute abundances (mu), and bias.em/bias.wls (e.g., to em_bias_estimates.csv). The log-scale linear regression framework with the sample-specific offset is analogous to a log-ratio transformation for handling compositionality.

## Related tools

- **ANCOM-BC** (Method whose bias-estimation step (offset term in the log-scale linear regression) is implemented by the E-M Gaussian mixture fit) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Implementation language for the E-M algorithm and the ANCOM-BC code archive) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (R-package dependency required for running the ANCOM-BC archive scripts) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (R-package dependency providing numerical optimization used by the archive implementation) — github.com/FrederikHuangLin/ANCOM-BC-Code-Archive
- **ANCOMBC** (Packaged R implementation exposing ancombc / ancombc2 with the same bias-correction machinery) — FrederickHuangLin/ANCOMBC

## Examples

```
ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method = "BH", tol.EM = 1e-5, max.iterNum = 100, perNum, alpha); bias <- res$bias.em
```

## Evaluation signals

- E-M iterations converged within max.iterNum = 100 iterations at tol.EM = 1e-5
- bias.em and bias.wls both represent the mean difference of log (natural log) sampling fractions between groups and should be in reasonable agreement, with WLS additionally providing the variance approximation
- Estimated sampling fractions d are returned on the natural log scale, and mu gives per-group log mean absolute abundances consistent with the fitted model
- Downstream res table: W = mean.difference/se and p.val from a two-sided Z-test, with q.val produced by the chosen adj.method
- Entries flagged as structural zeros (1s in struc.zero) contribute nothing to Delta_i used in the mixture fit

## Limitations

- ANCOM-BC may not perform well with very small sample sizes, such as n = 5 per group, and FDR is not controlled in such cases
- Slightly elevated FDR may occur when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var
- Pseudo-count handling differs across ANCOMBC package versions (before vs. from 2.10.1), affecting sensitivity-analysis results and the power/false-positive trade-off

## Evidence

- [intro] The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data"
- [readme] bias.em: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm.: "bias.em`: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm."
- [readme] tol.EM: Numeric. The iteration convergence tolerance for E-M algorithm.: "tol.EM`: Numeric. The iteration convergence tolerance for E-M algorithm."
- [discussion] it is the only method available in the literature that estimates the sampling fraction: "it is the only method available in the literature that estimates the sampling fraction"
- [readme] d`: A numeric vector. Estimated sampling fractions in log scale (natural log).: "d`: A numeric vector. Estimated sampling fractions in log scale (natural log)."
