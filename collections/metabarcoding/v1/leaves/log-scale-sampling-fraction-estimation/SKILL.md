---
name: log-scale-sampling-fraction-estimation
description: Use when you need to estimate or verify the sample-specific offset (bias)
  term in ANCOM-BC's log-scale linear regression, i.e., reconstruct how bias.em and
  bias.wls arise from the taxa-level Delta_i mixture fit.
when_to_use_negative:
- The analysis goal is differential abundance calling itself (mean.difference, W,
  p.val, q.val) rather than reconstructing or inspecting the bias-offset estimation
  step
- Very small sample sizes (e.g., n = 5 per group), where ANCOM-BC may not perform
  well and FDR is not controlled
- Data have not been pre-processed (no structural-zero matrix or grp.ind available)
  — run feature_table_pre_process first
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: ANCOM-BC
  role: Method whose ANCOM_BC function implements the E-M fit and returns bias.em,
    bias.wls, d, and mu
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: Required R library dependency for running the ANCOM-BC archive code
- name: nloptr
  role: Required R library dependency used by the ANCOM-BC archive code for optimization
    in the E-M step
- name: R
  role: Language in which the E-M reconstruction and bias estimation scripts are implemented
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:3686244c9270bddb082b887f60f379e42017ac154db454e9f8eb58ee0c9dae04
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
yes_rate: 0.5384615384615384
n_claims: 13
---

# log-scale-sampling-fraction-estimation

## Summary

Reconstructs ANCOM-BC's estimation of the sampling-fraction bias term: fitting a three-component Gaussian mixture to per-taxon log-scale mean differences via an E-M algorithm, and cross-checking with a weighted-least-squares variance approximation.

## When to use

Use when you need to estimate or verify the sample-specific offset (bias) term in ANCOM-BC's log-scale linear regression, i.e., reconstruct how bias.em and bias.wls arise from the taxa-level Delta_i mixture fit.

## When NOT to use

- The analysis goal is differential abundance calling itself (mean.difference, W, p.val, q.val) rather than reconstructing or inspecting the bias-offset estimation step
- Very small sample sizes (e.g., n = 5 per group), where ANCOM-BC may not perform well and FDR is not controlled
- Data have not been pre-processed (no structural-zero matrix or grp.ind available) — run feature_table_pre_process first

## Inputs

- Pre-processed OTU/taxon-by-sample feature table from feature_table_pre_process (e.g., dietswap absolute abundances)
- grp.name: character vector of levels of group.var
- grp.ind: numeric vector assigning each sample to a group level
- struc.zero: 0/1 matrix with 1 indicating a taxon is a structural zero in the corresponding group

## Outputs

- bias.em: estimated mean difference of log (natural log) sampling fractions between groups via E-M
- bias.wls: estimated mean difference of log sampling fractions via weighted least squares (variance approximation)
- d: numeric vector of estimated sampling fractions in natural-log scale
- mu: numeric vector of estimated log mean absolute abundance for each group
- em_bias_estimates.csv: saved d, mu, bias.em, and bias.wls values

## How to apply

Load the pre-processed taxa-by-sample feature table, group levels (grp.name), group assignments (grp.ind), and the structural-zero matrix (struc.zero) produced by feature_table_pre_process from the dietswap example data. Compute per-taxon differences Delta_i on the natural log scale between groups, masking entries flagged as structural zeros in struc.zero. Fit a three-component Gaussian mixture model to Delta_i with an E-M algorithm, iterating to tol.EM = 1e-5 with a maximum of max.iterNum = 100 iterations (via nloptr in R with dplyr). Extract bias.em, the estimated mean difference of log (natural log) sampling fractions between groups from the E-M step, and bias.wls, the weighted-least-squares estimate that provides the variance approximation. The offset term on the log scale is the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with compositionality.

## Related tools

- **ANCOM-BC** (Method whose ANCOM_BC function implements the E-M fit and returns bias.em, bias.wls, d, and mu) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (Required R library dependency for running the ANCOM-BC archive code)
- **nloptr** (Required R library dependency used by the ANCOM-BC archive code for optimization in the E-M step)
- **R** (Language in which the E-M reconstruction and bias estimation scripts are implemented)

## Examples

```
library(dplyr); library(nloptr); res <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="BH", tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05); res$bias.em; res$bias.wls
```

## Evaluation signals

- E-M converged within max.iterNum = 100 iterations at tol.EM = 1e-5
- bias.em and bias.wls both represent the mean difference of log (natural log) sampling fractions between groups
- bias.wls provides the weighted-least-squares variance approximation comparable to bias.em
- Structural-zero entries in struc.zero are masked out of Delta_i before fitting the Gaussian mixture
- Saved em_bias_estimates.csv contains d (sampling fractions), mu (per-group log mean absolute abundances), and both bias estimates

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group, and FDR is not controlled in such cases
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var
- perNum (permutations) is only active when there are more than 2 groups

## Evidence

- [intro] The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data"
- [readme] bias.em: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm.: "bias.em: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm."
- [readme] tol.EM: Numeric. The iteration convergence tolerance for E-M algorithm.: "tol.EM: Numeric. The iteration convergence tolerance for E-M algorithm."
- [discussion] it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions across samples: "it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions across samples"
- [abstract] ANCOM-BC), which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples.: "ANCOM-BC), which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples."
