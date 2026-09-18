---
name: reproducible-r-script-execution
description: Use when the goal is to reproduce results from a published microbiome
  DA study whose authors archived R scripts and a user manual (e.g. the ANCOM-BC Nat.
  Comm. paper), or to rerun documented R functions with paper-specified parameters.
when_to_use_negative:
- The target method is the maintained Bioconductor ANCOMBC package (ancombc/ancombc2)
  rather than the archived v1.0 scripts — the package API (formula/group arguments,
  prv_cut) differs from ANCOM_BC(feature.table, grp.name, grp.ind, ...)
- The group variable is continuous — ANCOM-BC v1.0 only supports a discrete group.var
- Sample sizes are very small (e.g. n = 5 per group), where the paper reports FDR
  is not controlled
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: R
  role: Runtime environment in which the archived ANCOM-BC scripts are executed
- name: dplyr
  role: Required library attached before running the archived scripts
- name: nloptr
  role: Required library attached before running the archived scripts (used by the
    E-M optimization)
- name: ANCOM-BC
  role: Method implemented by the archived scripts; run via ANCOM_BC() with tol.EM=1e-5,
    max.iterNum=100, bonferroni, alpha=0.05
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOMBC
  role: Corresponding maintained R package for the method, referenced for package-based
    use instead of archived code
  repo: github.com/FrederickHuangLin/ANCOMBC
- name: metagenomeSeq
  role: Competing DA method (ZIG) applied to the same simulated replicates in the
    FDR/power benchmark
- name: DESeq2
  role: Competing DA method applied with MED/UQ/TMM normalization variants in the
    benchmark
- name: edgeR
  role: Competing DA method applied with UQ/TMM variants in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:dc54e22bab0cba866c705396179c4c6620e2e4b390e2c3ef7b3b09a5f82164f8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM
  - ANCOM-BC
  - ANCOMBC
  - DESeq2
  - R
  - dplyr
  - edgeR
  - metagenomeSeq
  - nloptr
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 15
---

# reproducible-r-script-execution

## Summary

Execute archived R scripts from the ANCOM-BC Code Archive to reproduce published differential abundance benchmarks, loading the exact dependencies (dplyr, nloptr) declared in the repository and running documented functions with the paper's parameter settings.

## When to use

Use when the goal is to reproduce results from a published microbiome DA study whose authors archived R scripts and a user manual (e.g. the ANCOM-BC Nat. Comm. paper), or to rerun documented R functions with paper-specified parameters.

## When NOT to use

- The target method is the maintained Bioconductor ANCOMBC package (ancombc/ancombc2) rather than the archived v1.0 scripts — the package API (formula/group arguments, prv_cut) differs from ANCOM_BC(feature.table, grp.name, grp.ind, ...)
- The group variable is continuous — ANCOM-BC v1.0 only supports a discrete group.var
- Sample sizes are very small (e.g. n = 5 per group), where the paper reports FDR is not controlled

## Inputs

- Feature/OTU table (data frame or matrix, taxa in rows, samples in columns)
- Metadata data frame with sample IDs and a discrete group.var column
- Archived R scripts from github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- Simulated datasets generated under the Poisson-Gamma model (e.g. 100 replicates per design)

## Outputs

- ANCOM_BC result object: res data frame with mean.difference, se, W, p.val, q.val, diff.abn
- Estimated log-scale sampling fractions (d) and group means (mu), plus bias.em and bias.wls
- Pre-processed feature table, library.size, group.name, group.ind, and structure.zeros from feature_table_pre_process
- Aggregated FDR and power summary table (e.g. fdr_power_table.csv)

## How to apply

Clone the archived repository (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive) and attach the two R-package dependencies the README declares: library(dplyr) and library(nloptr). Follow the README's two-stage workflow: first preprocess the OTU table with feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb), then run the main function ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha). Use the paper's benchmark settings when reproducing simulations: tol.EM=1e-5, max.iterNum=100, adj.method='bonferroni', alpha=0.05, declaring a taxon differentially abundant when q.val < alpha. Capture the documented outputs (res with mean.difference, se, W, p.val, q.val, diff.abn; plus d, mu, bias.em, bias.wls) and write aggregated summaries (e.g. fdr_power_table.csv) for validation against the published figures.

## Related tools

- **R** (Runtime environment in which the archived ANCOM-BC scripts are executed)
- **dplyr** (Required library attached before running the archived scripts)
- **nloptr** (Required library attached before running the archived scripts (used by the E-M optimization))
- **ANCOM-BC** (Method implemented by the archived scripts; run via ANCOM_BC() with tol.EM=1e-5, max.iterNum=100, bonferroni, alpha=0.05) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOMBC** (Corresponding maintained R package for the method, referenced for package-based use instead of archived code) — github.com/FrederickHuangLin/ANCOMBC
- **metagenomeSeq** (Competing DA method (ZIG) applied to the same simulated replicates in the FDR/power benchmark)
- **DESeq2** (Competing DA method applied with MED/UQ/TMM normalization variants in the benchmark)
- **edgeR** (Competing DA method applied with UQ/TMM variants in the benchmark)

## Examples

```
library(dplyr); library(nloptr); out <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)
```

## Evaluation signals

- Scripts run without error after attaching exactly library(dplyr) and library(nloptr), the two dependencies the README requires
- ANCOM_BC returns a res data frame containing the documented columns mean.difference, se, W, p.val, q.val, and diff.abn
- diff.abn flags taxa exactly where q.val < alpha (0.05) under bonferroni adjustment
- Reproduced FDR/power table (fdr_power_table.csv) shows only ANCOM and ANCOM-BC at or below the 5% nominal FDR in both the small unbalanced (n1=20, n2=30) and large balanced (n1=n2=50) designs, while edgeR FDR reaches ~40% and metagenomeSeq (ZIG) up to ~70%

## Limitations

- ANCOM-BC may not perform well with very small sample sizes such as n = 5 per group, where FDR is not controlled
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 in the archive only supports a discrete group.var and cross-sectional two/multi-group comparisons

## Evidence

- [readme] The following libraries need to be included for the R code to run: "## R-package dependencies
The following libraries need to be included for the R code to run"
- [readme] ANCOM_BC usage and arguments: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [readme] q.val and diff.abn definition: "diff.abn. Logical. TRUE if the taxon has q.val less than alpha."
- [methods] P-value from two-sided Z-test: "P-value obtained from two-sided Z-test using the test statistic W"
- [abstract] ANCOM-BC controls FDR: "it (a) provides statistically valid test with appropriate p-values, (b) provides confidence intervals for differential abundance of each taxon, (c) controls the False Discovery Rate (FDR)"
