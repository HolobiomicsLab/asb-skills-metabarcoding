---
name: weighted-least-squares-variance-approximation
description: 'Use when reconstructing or validating ANCOM-BC''s bias-correction step:
  you have taxa-level log-scale mean differences Delta_i between groups and need the
  mean difference of log sampling fractions plus a variance approximation via weighted
  least squares.'
when_to_use_negative:
- The data have not been pre-processed and structural zeros not identified — WLS on
  raw compositional counts without masking struc.zero entries yields a biased delta
- Sample sizes are very small (e.g. n = 5 per group), where the article notes ANCOM-BC
  may not perform well and FDR is not controlled
- The goal is taxon-specific bias correction (differential sequencing efficiencies,
  as in SECOM/ANCOM-BC2) rather than the sample-specific sampling-fraction offset
  of ANCOM-BC v1.0
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: R
  role: Implementation language for the ANCOM-BC E-M and WLS bias-estimation code
- name: dplyr
  role: Required R library dependency for the ANCOM-BC-Code-Archive scripts that compute
    bias.em/bias.wls
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: Required R optimization library used by the ANCOM-BC estimation routines
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM-BC
  role: Method whose sampling-fraction bias term delta is estimated; the main function
    ANCOM_BC returns both bias.em and bias.wls
  repo: https://github.com/FrederickHuangLin/ANCOMBC
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:2438bf21af91a8384734bf254a3fb77e29f012087c0ab9d115822170bc71b5d3
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# weighted-least-squares-variance-approximation

## Summary

A weighted least squares (WLS) estimator of the sampling-fraction bias term delta in ANCOM-BC, providing both a point estimate of the mean difference of log sampling fractions between groups and a variance approximation that can be benchmarked against the E-M estimate (bias.em).

## When to use

Use when reconstructing or validating ANCOM-BC's bias-correction step: you have taxa-level log-scale mean differences Delta_i between groups and need the mean difference of log sampling fractions plus a variance approximation via weighted least squares.

## When NOT to use

- The data have not been pre-processed and structural zeros not identified — WLS on raw compositional counts without masking struc.zero entries yields a biased delta
- Sample sizes are very small (e.g. n = 5 per group), where the article notes ANCOM-BC may not perform well and FDR is not controlled
- The goal is taxon-specific bias correction (differential sequencing efficiencies, as in SECOM/ANCOM-BC2) rather than the sample-specific sampling-fraction offset of ANCOM-BC v1.0

## Inputs

- Pre-processed OTU/feature table (taxa in rows, samples in columns) from feature_table_pre_process
- grp.name: character vector of group levels of group.var
- grp.ind: numeric vector of per-sample group assignments
- struc.zero: 0/1 matrix flagging structural zeros per taxon per group
- Group-level statistics (taxon variances) derived from the feature table for weighting the WLS fit

## Outputs

- bias.wls: numeric estimate of the mean difference of log (natural log) sampling fractions between groups via weighted least squares, with variance approximation
- bias.em: E-M counterpart (mean difference of log sampling fractions) for cross-checking
- d: numeric vector of estimated sampling fractions in log scale (natural log)
- mu: numeric vector of estimated log mean absolute abundance per group
- em_bias_estimates.csv: table saving d, mu, and bias.em/bias.wls

## How to apply

Load the pre-processed taxa-by-sample feature table, group levels (grp.name), group assignments (grp.ind), and structural-zero matrix produced by feature_table_pre_process (e.g. from the dietswap dataset). Compute taxa-level differences Delta_i on the natural log scale between groups, masking entries flagged as structural zeros in struc.zero. Fit the three-component Gaussian mixture model to Delta_i with the E-M algorithm (tol.EM = 1e-5, max.iterNum = 100) to obtain bias.em, then compute the WLS estimate bias.wls — the mean difference of log (natural log) sampling fractions between groups — whose weighted formulation supplies the variance approximation of the bias. Compare bias.wls against bias.em to confirm both represent the same estimand; bias.wls provides the closed-form variance used downstream for the standard error and Z-test statistic W = mean.difference/se. R implementations rely on dplyr and nloptr, per the ANCOM-BC-Code-Archive dependencies.

## Related tools

- **R** (Implementation language for the ANCOM-BC E-M and WLS bias-estimation code)
- **dplyr** (Required R library dependency for the ANCOM-BC-Code-Archive scripts that compute bias.em/bias.wls) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (Required R optimization library used by the ANCOM-BC estimation routines) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM-BC** (Method whose sampling-fraction bias term delta is estimated; the main function ANCOM_BC returns both bias.em and bias.wls) — https://github.com/FrederickHuangLin/ANCOMBC

## Examples

```
ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM = 1e-5, max.iterNum = 100, perNum, alpha)
```

## Evaluation signals

- E-M convergence: iterations stop within max.iterNum = 100 at tol.EM = 1e-5
- bias.wls and bias.em both estimate the same quantity — the mean difference of log (natural log) sampling fractions between groups — and should be in close agreement
- Structural zeros flagged in struc.zero are excluded (masked) from Delta_i before fitting
- Output d is interpretable as sampling fractions in log scale (natural log), and mu as per-group log mean absolute abundances
- Downstream W = mean.difference/se is finite and the two-sided Z-test yields valid p-values

## Limitations

- ANCOM-BC v1.0 only supports discrete group.var; the WLS/E-M bias estimation is defined for cross-sectional two-group (or multi-group with permutation) comparisons
- The methodology may not perform well with very small sample sizes, such as n = 5 per group
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large

## Evidence

- [readme] bias.wls: "bias.wls`: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through weighted least squares."
- [readme] bias.em: "bias.em`: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm."
- [methods] E-M parameters: "The iteration convergence tolerance for E-M algorithm"
- [intro] offset as bias correction: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data"
- [discussion] small-sample caveat: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
