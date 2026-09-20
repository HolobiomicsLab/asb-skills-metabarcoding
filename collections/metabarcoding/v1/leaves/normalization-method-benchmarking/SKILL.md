---
name: normalization-method-benchmarking
description: Use when simulated or ground-truth microbiome count data with known sampling
  fractions is available and you must quantify which normalization method removes
  group-wise bias in sampling fractions.
when_to_use_negative:
- Input is real observational data with no ground-truth sampling fractions, so residuals
  cannot be computed — use the method's DA testing output instead
- Goal is differential abundance testing itself rather than evaluating normalization
  performance
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC-based estimation
  may be unreliable
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: Primary method that estimates sampling fractions in log scale via E-M algorithm;
    its estimates anchor the residual comparison
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: metagenomeSeq
  role: Provides CSS normalization used as a competing method in the benchmark
- name: DESeq2
  role: Provides MED normalization used as a competing method in the benchmark
- name: edgeR
  role: Provides modified UQ and TMM normalization used as competing methods in the
    benchmark
- name: dplyr
  role: R library dependency for running the ANCOM-BC archive scripts
- name: nloptr
  role: R library dependency for the optimization in the ANCOM-BC E-M algorithm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:799af8566f2d00198f3d7a6d63a88cfe70ad8e345425bd300a0d775403e2577f
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
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# normalization-method-benchmarking

## Summary

Benchmark microbiome normalization methods (ANCOM-BC, ELib-UQ, ELib-TMM, CSS, MED, UQ, TMM, TSS) by comparing per-sample residuals between estimated and true sampling fractions on simulated count data, as done for Fig. 3 of the ANCOM-BC paper.

## When to use

Use when simulated or ground-truth microbiome count data with known sampling fractions is available and you must quantify which normalization method removes group-wise bias in sampling fractions.

## When NOT to use

- Input is real observational data with no ground-truth sampling fractions, so residuals cannot be computed — use the method's DA testing output instead
- Goal is differential abundance testing itself rather than evaluating normalization performance
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC-based estimation may be unreliable

## Inputs

- Simulated OTU/feature count table (taxa in rows, samples in columns) with known true sampling fractions and two experimental group labels
- Metadata data frame with sample IDs and group.var
- R environment with dplyr, nloptr, ANCOM-BC archive scripts, metagenomeSeq, DESeq2, edgeR

## Outputs

- Per-sample sampling-fraction residuals (natural-log scale) for each of the eight normalization methods
- method_residual_variances.csv with one residual variance per method
- fig3_residual_boxplot.png, a ggplot2-style box plot of residual distributions across the eight methods

## How to apply

Generate simulation data (e.g., unbalanced microbial load with balanced library size) from the ANCOM-BC Code Archive scripts in R with dplyr and nloptr, and pre-process the feature table with feature_table_pre_process using zero.cut, lib.cut, and neg.lb (asymptotic lower bounds flag structural zeros). Run ANCOM_BC with the E-M algorithm (tol.EM = 1e-5, max.iterNum = 100) to obtain estimated sampling fractions in natural-log scale (vector d) and the estimated mean difference of log sampling fractions (bias.em). Apply competing methods to the same simulated counts: CSS from metagenomeSeq, MED from DESeq2, and modified UQ and TMM from edgeR, alongside ELib-UQ, ELib-TMM, and TSS. For each method, compute per-sample residuals as the deviation between observed and estimated log sampling fractions, summarize residual variances, and compare residual distributions (e.g., box plots) across methods, checking whether residuals intermix between the two experimental groups rather than clustering by group label.

## Related tools

- **ANCOM-BC** (Primary method that estimates sampling fractions in log scale via E-M algorithm; its estimates anchor the residual comparison) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **metagenomeSeq** (Provides CSS normalization used as a competing method in the benchmark)
- **DESeq2** (Provides MED normalization used as a competing method in the benchmark)
- **edgeR** (Provides modified UQ and TMM normalization used as competing methods in the benchmark)
- **dplyr** (R library dependency for running the ANCOM-BC archive scripts)
- **nloptr** (R library dependency for the optimization in the ANCOM-BC E-M algorithm)

## Examples

```
source('ANCOM_BC.R'); pre <- feature_table_pre_process(feature.table, meta.data, 'sample', 'group', zero.cut=0.1, lib.cut=1000, neg.lb=TRUE); out <- ANCOM_BC(pre$feature.table, pre$group.name, pre$group.ind, pre$structure.zeros, 'BH', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)
```

## Evaluation signals

- fig3_residual_boxplot.png exists and contains box plots for all eight methods (ANCOM-BC, ELib-UQ, ELib-TMM, CSS, MED, UQ, TMM, TSS)
- method_residual_variances.csv lists exactly one residual variance per method, consistent with the plotted distributions
- ANCOM-BC residuals show the smallest variance among the eight methods
- ANCOM-BC-normalized residuals are intermixed between the two groups without clustering by group label
- Existing methods (except UQ and TMM) show systematically separated residuals between the two groups in the large sampling-fraction variability scenario

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group; FDR is not controlled in such cases
- Slightly elevated FDR may occur when the proportion of differentially abundant taxa is too large
- Benchmark conclusions are grounded in the specific simulation scenario (unbalanced microbial load, balanced library size, large sampling-fraction variability) and may not transfer to other designs

## Evidence

- [discussion] it is the only method available in the literature that estimates the sampling fraction: "ANCOM-BC enjoys several important unique characteristics. First, it is the only method available in the literature that estimates the sampling fraction"
- [methods] Taxa with proportion of zeroes greater than zero.cut are not included: "zero.cut: Numerical fraction between 0 and 1. Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [methods] Samples with library size less than lib.cut are not included: "lib.cut: Numeric. Samples with library size less than lib.cut are not included in the analysis"
- [readme] Estimated sampling fractions in log scale (natural log): "d: A numeric vector. Estimated sampling fractions in log scale (natural log)."
- [methods] The iteration convergence tolerance for E-M algorithm: "tol.EM: Numeric. The iteration convergence tolerance for E-M algorithm"
