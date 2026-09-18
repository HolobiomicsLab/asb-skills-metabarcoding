---
name: compositional-data-bias-correction
description: Use when analyzing relative abundance OTU/feature tables where sampling
  fractions may differ across experimental groups and absolute-abundance differential
  abundance tests with p-values, confidence intervals, and FDR control are needed.
when_to_use_negative:
- Sample sizes are very small (e.g. n = 5 per group) — ANCOM-BC may not perform well
  and FDR is not controlled in such cases
- The proportion of differentially abundant taxa is too large — FDR may be slightly
  elevated
- The taxon of interest is a structural zero in a group — ANCOM-BC is not designed
  to detect significant abundance differences for structural zeros
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: ANCOM-BC
  role: 'Core method: estimates sampling fractions via E-M algorithm and performs
    bias-corrected two-group or multi-group DA analysis'
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: R library dependency required for the ANCOM-BC scripts
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: R library dependency required for the ANCOM-BC scripts
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: metagenomeSeq
  role: Competing normalization/DA method used for comparison against ANCOM-BC (CSS
    normalization)
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: DESeq2
  role: Competing method providing Median (MED) normalization for comparison against
    ANCOM-BC
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: edgeR
  role: Source of modified UQ and TMM normalization methods used as competing baselines
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
merged_aliases:
- compositional-bias-correction-offset-modeling
merged_alias_records:
- alias: compositional-bias-correction-offset-modeling
  slug: compositional-bias-correction-offset-modeling
  cosine_score: 0.826
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_005
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:c4222a7fb6383dda3b0b8f3ed1a42884c227edb522d39c79e0f7ac1223ab0ae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - DESeq2
  - R
  - dplyr
  - edgeR
  - metagenomeSeq
  - nloptr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# compositional-data-bias-correction

## Summary

ANCOM-BC differential abundance analysis for compositional microbiome data: it estimates sample-specific sampling fractions in log scale and corrects the bias introduced by their differences across samples, yielding per-taxon mean differences, standard errors, Z-test p-values, and adjusted q-values for absolute abundance.

## When to use

Use when analyzing relative abundance OTU/feature tables where sampling fractions may differ across experimental groups and absolute-abundance differential abundance tests with p-values, confidence intervals, and FDR control are needed.

## When NOT to use

- Sample sizes are very small (e.g. n = 5 per group) — ANCOM-BC may not perform well and FDR is not controlled in such cases
- The proportion of differentially abundant taxa is too large — FDR may be slightly elevated
- The taxon of interest is a structural zero in a group — ANCOM-BC is not designed to detect significant abundance differences for structural zeros

## Inputs

- OTU/feature table (data frame or matrix) with taxa in rows and samples in columns
- Sample metadata data frame with a sample ID column and a discrete group.var (e.g. nationality, bmi_group)
- Parameter settings: zero.cut, lib.cut, neg.lb, adj.method, tol.EM, max.iterNum, perNum, alpha

## Outputs

- Result data frame per taxon: mean.difference, se, W, p.val, q.val, diff.abn
- Estimated sampling fractions in log scale (d) and estimated log mean absolute abundance per group (mu)
- Estimated mean difference of log sampling fractions via E-M (bias.em) and weighted least squares (bias.wls)
- Pre-processed feature table, library.size, group.name, group.ind, and structure.zeros matrix

## How to apply

Preprocess the observed OTU table (taxa in rows, samples in columns) with feature_table_pre_process using zero.cut = 0.90 to drop taxa with >90% zeroes, lib.cut = 1000 to exclude samples with library size <1000, and neg.lb = TRUE to classify structural zeros via each taxon's asymptotic lower bound per group. Run the two-group ANCOM_BC function with adj.method = 'bonferroni', tol.EM = 1e-5, max.iterNum = 100, perNum = 1000, alpha = 0.05; it uses an E-M algorithm to estimate the log-scale (natural log) sampling fractions and their mean difference between groups. The method recasts DA analysis as a linear regression problem in log scale with an offset term for the sampling fraction, which is analogous to a log-ratio transformation for dealing with compositionality. Interpret outputs per taxon: mean.difference (log mean difference of absolute abundance), se, W = mean.difference/se (two-sided Z-test statistic), p.val, q.val, and diff.abn = TRUE when q.val < alpha; the output also provides d (estimated log sampling fractions), mu (estimated log mean absolute abundance per group), bias.em, and bias.wls. Validate results by confirming taxa flagged diff.abn = TRUE reproduce known group differences, e.g. Verrucomicrobia significantly increased in US relative to Malawi/Venezuela samples, and residuals of estimated log sampling fractions are intermixed between groups rather than clustered by group label.

## Related tools

- **ANCOM-BC** (Core method: estimates sampling fractions via E-M algorithm and performs bias-corrected two-group or multi-group DA analysis) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (R library dependency required for the ANCOM-BC scripts) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (R library dependency required for the ANCOM-BC scripts) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **metagenomeSeq** (Competing normalization/DA method used for comparison against ANCOM-BC (CSS normalization)) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **DESeq2** (Competing method providing Median (MED) normalization for comparison against ANCOM-BC) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **edgeR** (Source of modified UQ and TMM normalization methods used as competing baselines) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Examples

```
out <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="bonferroni", tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)
```

## Evaluation signals

- diff.abn = TRUE per taxon must correspond to bonferroni-adjusted q.val < alpha = 0.05
- W must equal mean.difference / se (test statistic of the two-sided Z-test)
- After ANCOM-BC normalization, sampling-fraction residuals should be intermixed between the two experimental groups without clustering by group label, and show the smallest variance among compared methods
- Preprocessing must drop taxa with proportion of zeroes > zero.cut (0.90) and samples with library size < lib.cut (1000)
- Reproduced findings must match the reported biology: e.g. Verrucomicrobia significantly increased in absolute abundance in US relative to Malawi adults/infants and Venezuelan adults, and Bacteroidetes/Firmicutes log ratio larger in Malawi and Venezuela infants than US infants

## Limitations

- May not perform well when sample sizes are very small, such as n = 5 per group; FDR is not controlled in such cases
- May have slightly elevated FDR when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var (no continuous covariates in the archived function)
- Standard methods like ANOVA and Kruskal–Wallis that do not account for compositionality show inflated FDR, so this skill should replace rather than complement them for compositional DA testing

## Evidence

- [abstract] ANCOM-BC), which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples.: "ANCOM-BC), which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples."
- [methods] Taxa with proportion of zeroes greater than zero.cut are not included in the analysis: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [readme] mean.difference: Numeric. The estimated mean difference of absolute abundance between groups in log scale (natural log);: "mean.difference: Numeric. The estimated mean difference of absolute abundance between groups in log scale (natural log);"
- [methods] P-value obtained from two-sided Z-test using the test statistic W: "P-value obtained from two-sided Z-test using the test statistic W"
- [intro] The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality"
