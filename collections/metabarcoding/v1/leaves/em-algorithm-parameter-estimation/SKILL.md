---
name: em-algorithm-parameter-estimation
description: Use when performing differential abundance testing on compositional microbiome
  count data (OTU/feature tables) where bias from differing sampling fractions across
  samples must be estimated and corrected, as in ANCOM-BC.
when_to_use_negative:
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC may not perform
  well and FDR is not controlled
- The group variable is continuous — ANCOM-BC v1.0 only supports a discrete group.var
- The goal is testing mean relative abundance rather than mean absolute abundance,
  which is not equivalent for compositional data
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ANCOM-BC
  role: Provides feature_table_pre_process and the ANCOM_BC E-M fitting routine that
    estimates sampling-fraction bias and returns per-taxon DA statistics
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: R dependency used in the implementation of the ANCOM-BC estimation routine
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: R dependency for data wrangling of the OTU table and metadata before and after
    E-M fitting
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Execution environment; required libraries dplyr, nloptr, and the ANCOMBC package
  repo: https://github.com/FrederickHuangLin/ANCOMBC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:891ce868db35d195c0b12696feee189da63752554a9799727f412277b2ccf632
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# em-algorithm-parameter-estimation

## Summary

Fits the ANCOM-BC log-linear model of absolute abundance with a sample-specific sampling-fraction offset via an E-M algorithm, iteratively estimating bias until convergence (tol.EM) or a maximum number of iterations (max.iterNum), yielding bias-corrected mean differences, standard errors, W statistics, and p-values per taxon.

## When to use

Use when performing differential abundance testing on compositional microbiome count data (OTU/feature tables) where bias from differing sampling fractions across samples must be estimated and corrected, as in ANCOM-BC.

## When NOT to use

- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC may not perform well and FDR is not controlled
- The group variable is continuous — ANCOM-BC v1.0 only supports a discrete group.var
- The goal is testing mean relative abundance rather than mean absolute abundance, which is not equivalent for compositional data

## Inputs

- Absolute abundance OTU/feature table (taxa in rows, samples in columns)
- Sample metadata data frame with sample IDs and a discrete group.var (e.g., nationality, bmi_group)
- Pre-processing outputs: feature.table, library.size, group.name, group.ind, structure.zeros from feature_table_pre_process
- Settings: adj.method ('holm','hochberg','hommel','bonferroni','BH','BY','fdr','none'), tol.EM, max.iterNum, perNum, alpha

## Outputs

- res data frame per taxon: mean.difference, se, W, p.val, q.val, diff.abn
- d: estimated sampling fractions in log scale (natural log)
- mu: estimated log (natural log) mean absolute abundance for each group
- bias.em and bias.wls: estimated mean difference of log sampling fractions between groups via E-M and weighted least squares
- Results CSV (e.g., supptable3_log_ratio.csv) assembled from per-taxon statistics

## How to apply

Preprocess the absolute-abundance feature table with feature_table_pre_process (e.g., sample.var='sample', group.var='nationality', zero.cut, lib.cut, neg.lb=TRUE) and feed the retained feature.table, grp.name, grp.ind, and structural-zero matrix into ANCOM_BC. Run the E-M algorithm with tol.EM = 1e-5, max.iterNum = 100, perNum = 1000, adj.method = 'bonferroni', and alpha = 0.05; the E-M step estimates the mean difference of log (natural log) sampling fractions between groups (bias.em), which serves as the offset/bias correction in the log-scale linear regression framework. For each taxon, extract mean.difference (log-scale difference of absolute abundance), se, W = mean.difference/se, p.val from a two-sided Z-test, and q.val adjusted by the chosen method. Declare a taxon differentially abundant when diff.abn is TRUE (q.val < alpha). The estimated log sampling fractions (d) and per-group log mean absolute abundances (mu) are also returned for downstream log-ratio analyses.

## Related tools

- **ANCOM-BC** (Provides feature_table_pre_process and the ANCOM_BC E-M fitting routine that estimates sampling-fraction bias and returns per-taxon DA statistics) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (R dependency used in the implementation of the ANCOM-BC estimation routine) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (R dependency for data wrangling of the OTU table and metadata before and after E-M fitting) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Execution environment; required libraries dplyr, nloptr, and the ANCOMBC package) — https://github.com/FrederickHuangLin/ANCOMBC

## Examples

```
out <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)
```

## Evaluation signals

- Convergence: E-M iterations stop when the change falls below tol.EM (set 1e-5) or max.iterNum (set 100) is reached
- W equals mean.difference divided by se for every taxon in the res output
- p.val corresponds to a two-sided Z-test on W; q.val equals adj.method applied to p.val (bonferroni in the exemplar run)
- diff.abn is TRUE exactly when q.val < alpha (0.05)
- Reproducing the Fig. 6b Bacteroidetes/Firmicutes log-ratio comparison on the global gut microbiota dataset should show the ratio larger among Malawi infants and significantly larger among Venezuela infants than US infants

## Limitations

- ANCOM-BC may not perform well with very small sample sizes (e.g., n = 5 per group); FDR is not controlled in such cases
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large
- perNum only takes effect when there exist more than 2 groups
- Structural-zero taxa are detected only via the presence/absence test and are excluded from the primary results

## Evidence

- [readme] EM tolerance: "tol.EM`: Numeric. The iteration convergence tolerance for E-M algorithm."
- [readme] EM iterations: "max.iterNum`: Numeric. The maximum number of iterations for E-M algorithm."
- [readme] bias estimate: "bias.em`: Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm."
- [methods] Z-test: "P-value obtained from two-sided Z-test using the test statistic W"
- [intro] offset rationale: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data"
