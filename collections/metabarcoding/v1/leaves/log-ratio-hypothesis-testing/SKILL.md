---
name: log-ratio-hypothesis-testing
description: Use when comparing the log ratio of two taxa (e.g., Bacteroidetes to
  Firmicutes) between groups such as US vs non-US subjects in compositional microbiome
  data, requiring bias-corrected mean differences, standard errors, and p-values.
when_to_use_negative:
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC may not perform
  well and FDR is not controlled
- The taxon of interest is classified as a structural zero in a group (neg.lb = TRUE
  flags it), since ANCOM-BC is not designed to test differential abundance of structural
  zeros
- The group variable is not discrete, since ANCOM-BC v1.0 only supports discrete group.var
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: ANCOM-BC
  role: 'Core method: fits the bias-corrected linear regression with sampling-fraction
    offset and returns mean.difference, se, W, p.val, q.val, and mu per taxon'
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Implementation language for preprocessing, ANCOM-BC execution, and CSV export
- name: dplyr
  role: Data manipulation library required by the ANCOM-BC code archive scripts
- name: nloptr
  role: Numerical optimization library required by the ANCOM-BC code archive scripts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:0fe6bc4ca4312b0d9539a91eeeadd085dff1ce4b79a3ee66ed609c16a39619ea
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# log-ratio-hypothesis-testing

## Summary

Tests hypotheses about log-ratios of microbial taxon abundances between experimental groups using ANCOM-BC's bias-corrected linear regression framework, which estimates sample-specific sampling fractions to correct compositional bias before testing.

## When to use

Use when comparing the log ratio of two taxa (e.g., Bacteroidetes to Firmicutes) between groups such as US vs non-US subjects in compositional microbiome data, requiring bias-corrected mean differences, standard errors, and p-values.

## When NOT to use

- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC may not perform well and FDR is not controlled
- The taxon of interest is classified as a structural zero in a group (neg.lb = TRUE flags it), since ANCOM-BC is not designed to test differential abundance of structural zeros
- The group variable is not discrete, since ANCOM-BC v1.0 only supports discrete group.var

## Inputs

- Phyloseq object or OTU table (taxa in rows, samples in columns) of absolute abundances
- Sample metadata data frame with sample IDs and discrete group variable (e.g., nationality, age group)
- Taxonomy table for identifying taxa (e.g., Bacteroidetes, Firmicutes)

## Outputs

- Results data frame with columns taxon, mean.difference, se, W, p.val, q.val, diff.abn
- Estimated log mean absolute abundances (mu) per group per taxon
- CSV file (e.g., supptable3_log_ratio.csv) with log-ratio test results
- Estimated sampling fractions in log scale (vector d)

## How to apply

Load the phyloseq object (e.g., the global gut microbiota dataset), extract the absolute abundance OTU table, sample metadata, and taxonomy table, then preprocess with feature_table_pre_process using sample.var='sample', the nationality/age group variable, zero.cut = 0.90, lib.cut = 1000, and neg.lb = TRUE, retaining feature.table, group.name, group.ind, and structure.zeros. Run ANCOM_BC with adj.method = 'bonferroni', tol.EM = 1e-5, max.iterNum = 100, perNum = 1000, and alpha = 0.05 to obtain per-taxon mean.difference, se, W statistic, p.val, and q.val. Extract the natural-log mean absolute abundances (mu) for the two taxa of interest (e.g., Bacteroidetes and Firmicutes) in each group from the fitted model. Compute the mean log ratio per stratum and test its equality across strata with the two-sided Z-test statistic W = mean.difference/se, comparing Malawi and Venezuela infants against US infants. Assemble the results table (taxon, mean.difference, se, W, p.val, q.val, diff.abn) and write it via write_csv (e.g., supptable3_log_ratio.csv).

## Related tools

- **ANCOM-BC** (Core method: fits the bias-corrected linear regression with sampling-fraction offset and returns mean.difference, se, W, p.val, q.val, and mu per taxon) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Implementation language for preprocessing, ANCOM-BC execution, and CSV export)
- **dplyr** (Data manipulation library required by the ANCOM-BC code archive scripts)
- **nloptr** (Numerical optimization library required by the ANCOM-BC code archive scripts)

## Examples

```
res <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)
```

## Evaluation signals

- Output table contains all required columns (taxon, mean.difference, se, W, p.val, q.val, diff.abn) and W equals mean.difference divided by se for each taxon
- q.val equals p.val adjusted by the specified method (e.g., 'bonferroni'), and diff.abn is TRUE exactly when q.val < alpha (0.05)
- mu values are natural-log mean absolute abundances, and the computed log ratio for Malawi/Venezuela infants is compared against US infants with the two-sided Z-test statistic W
- Preprocessing respects thresholds: taxa with proportion of zeroes > zero.cut (0.90) and samples with library size < lib.cut (1000) are excluded
- Result reproduces the direction of Fig. 6b: Bacteroidetes-to-Firmicutes ratio larger among Malawi infants than US infants and significantly larger among Venezuela infants

## Limitations

- ANCOM-BC may not perform well with very small sample sizes, such as n = 5 per group, and FDR is not controlled in such cases
- Slightly elevated FDR may occur when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var and two-group comparisons; perNum (permutations) is active only when there exist more than 2 groups

## Evidence

- [methods] W test statistic from ANCOM-BC output: "W: Numeric. mean.difference/se, which is the test statistic of ANCOM-BC."
- [methods] P-value from two-sided Z-test: "P-value obtained from two-sided Z-test using the test statistic W"
- [readme] Structural zeros are excluded from primary DA testing: [evidence span withheld — anchor status: not_found]
- [intro] Sampling-fraction offset serves as bias correction analogous to log-ratio transformation: "The offset term serves as the bias correction, and the linear regression framework in log scale is analogous to log-ratio transformation to deal with the compositionality of microbiome data"
- [intro] Testing mean relative abundance differs from testing mean absolute abundance: "Testing hypotheses regarding mean relative abundance is not equivalent to testing hypotheses regarding mean absolute abundance"
