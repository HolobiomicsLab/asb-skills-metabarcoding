---
name: microbiome-differential-abundance-analysis
description: Use when analyzing microbiome OTU/feature count tables to identify taxa
  differentially abundant across experimental groups (e.g., nationality, age strata,
  bmi_group) while accounting for compositionality and unequal sampling fractions.
when_to_use_negative:
- Very small sample sizes (e.g., n = 5 per group) — ANCOM-BC may not perform well
  and FDR is not controlled in such cases
- When the proportion of differentially abundant taxa is too large, FDR may be slightly
  elevated — consider alternatives or interpret q.vals cautiously
- Testing hypotheses about mean relative abundance rather than absolute abundance
  — the method targets absolute abundance via sampling-fraction correction
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: 'Core DA method: estimates sampling fractions, fits the bias-corrected log-scale
    linear model, and returns W, p.val, q.val, diff.abn per taxon'
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOMBC
  role: R package implementation of ANCOM-BC (ancombc) and related DA/correlation
    methods for microbiome data
  repo: FrederickHuangLin/ANCOMBC
- name: R
  role: Runtime environment; the ANCOM-BC Code Archive scripts are R code using phyloseq
    objects and write_csv export
- name: dplyr
  role: Required R dependency for data manipulation in the ANCOM-BC Code Archive scripts
- name: nloptr
  role: Required R dependency used by the ANCOM-BC scripts (optimization underlying
    model fitting)
merged_aliases:
- differential-abundance-analysis-of-microbiome-taxa
merged_alias_records:
- alias: differential-abundance-analysis-of-microbiome-taxa
  slug: differential-abundance-analysis-of-microbiome-taxa
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:7337e490d67a20d3f8dbc7a2fa793edf22cb8be76225d40c7133ba88e82387ca
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# microbiome-differential-abundance-analysis

## Summary

Performs bias-corrected differential abundance (DA) analysis of microbiome compositional data using ANCOM-BC, which estimates sample-specific sampling fractions and includes them as an offset in a log-scale linear regression to test absolute abundance differences between groups.

## When to use

Use when analyzing microbiome OTU/feature count tables to identify taxa differentially abundant across experimental groups (e.g., nationality, age strata, bmi_group) while accounting for compositionality and unequal sampling fractions.

## When NOT to use

- Very small sample sizes (e.g., n = 5 per group) — ANCOM-BC may not perform well and FDR is not controlled in such cases
- When the proportion of differentially abundant taxa is too large, FDR may be slightly elevated — consider alternatives or interpret q.vals cautiously
- Testing hypotheses about mean relative abundance rather than absolute abundance — the method targets absolute abundance via sampling-fraction correction

## Inputs

- OTU/feature abundance table (data frame or matrix, taxa in rows, samples in columns; absolute counts)
- Sample metadata data frame including sample IDs, the grouping variable (e.g., nationality, bmi_group, age strata), and covariates
- Taxonomy table for aggregating to phylum level (e.g., phyloseq object)

## Outputs

- Result table per taxon: mean.difference, se, W, p.val, q.val, diff.abn (e.g., supptable3_log_ratio.csv)
- Estimated log sampling fractions (d) and log mean absolute abundances per group (mu)
- Structural zeros matrix and pre-processed feature table from feature_table_pre_process
- Plots of estimated mean differences of absolute abundance (natural log) with confidence intervals per taxon

## How to apply

Load the OTU table (taxa in rows, samples in columns) and sample metadata, then run feature_table_pre_process with sample.var, group.var, zero.cut (e.g., 0.90 to drop taxa with >90% zeroes), lib.cut (e.g., 1000 to exclude low-library-size samples), and neg.lb=TRUE to flag structural zeros via the asymptotic lower bound. Run ANCOM_BC with adj.method (e.g., 'bonferroni'), tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05; the method models absolute abundance in a linear regression framework on the natural log scale with an offset term correcting for differing sampling fractions. Extract per-taxon mean.difference, se, W (mean.difference/se), p.val (two-sided Z-test), q.val (adjusted), diff.abn, plus the estimated log sampling fractions d and per-group log mean absolute abundances mu. For multi-group data, either run pairwise two-group comparisons or use stratified analyses (e.g., infant ≤2y vs adult 18–40y strata), and log-ratio contrasts (e.g., Bacteroidetes/Firmicutes) can be tested with the W statistic. Export the results table (taxon, mean.difference, se, W, p.val, q.val, diff.abn) via write_csv and visualize mean differences with confidence intervals, highlighting taxa with q.val < 0.05.

## Related tools

- **ANCOM-BC** (Core DA method: estimates sampling fractions, fits the bias-corrected log-scale linear model, and returns W, p.val, q.val, diff.abn per taxon) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOMBC** (R package implementation of ANCOM-BC (ancombc) and related DA/correlation methods for microbiome data) — FrederickHuangLin/ANCOMBC
- **R** (Runtime environment; the ANCOM-BC Code Archive scripts are R code using phyloseq objects and write_csv export)
- **dplyr** (Required R dependency for data manipulation in the ANCOM-BC Code Archive scripts)
- **nloptr** (Required R dependency used by the ANCOM-BC scripts (optimization underlying model fitting))

## Examples

```
feature.table = otu_absolute; sample.var = "sample"; group.var = "nationality"; pre.proc = feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut = 0.90, lib.cut = 1000, neg.lb = TRUE); out = ANCOM_BC(pre.proc$feature.table, pre.proc$group.name, pre.proc$group.ind, pre.proc$structure.zeros, adj.method = "bonferroni", tol.EM = 1e-5, max.iterNum = 100, perNum = 1000, alpha = 0.05)
```

## Evaluation signals

- Taxa flagged diff.abn = TRUE correspond to bonferroni-adjusted q.val < alpha = 0.05 and reproduce reported findings (e.g., Verrucomicrobia significantly increased in US relative to Malawi adults/infants and Venezuelan adults)
- Each result row carries the full schema: taxon, mean.difference, se, W (mean.difference/se), p.val, q.val, diff.abn
- Pre-processing filters behaved as specified: taxa with zero proportion > zero.cut removed, samples with library size < lib.cut excluded, structural zeros identified via neg.lb asymptotic lower bound
- E-M iteration converged within max.iterNum=100 at tol.EM=1e-5; W equals mean.difference divided by se
- Stratified log-ratio results (Bacteroidetes/Firmicutes) match the reported direction: ratio larger among Malawi infants than US, significantly larger among Venezuela infants than US

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group; FDR is not controlled in such cases
- May have slightly elevated FDR when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var; taxa classified as structural zeros are not suitable for the standard abundance test (handled via the presence/absence framework in newer package versions)
- perNum (permutation number) is only active when there exist more than 2 groups

## Evidence

- [readme] feature_table_pre_process removes high-zero taxa and small-library samples: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis."
- [methods] two-sided Z-test p-value from W statistic: "P-value obtained from two-sided Z-test using the test statistic W"
- [intro] offset term as bias correction in log-scale regression: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data"
- [discussion] ANCOM-BC uniquely estimates the sampling fraction: "it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions across samples"
- [readme] structural zero classification via neg.lb: "TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound."
