---
name: residual-distribution-boxplot-visualization
description: Use when comparing normalization methods on simulated count data with
  known sampling fractions and you need to show, per sample, the residual between
  observed and method-estimated log sampling fractions as a grouped box plot.
when_to_use_negative:
- When the data are raw observed counts with no known (true) sampling fractions —
  residuals against truth cannot be computed, so only real-data DA testing is appropriate,
  not this residual comparison
- When sample sizes are very small (e.g. n = 5 per group), where the article notes
  ANCOM-BC may not perform well and FDR is not controlled
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: ANCOM-BC
  role: Estimates sampling fractions in log scale (output d) and the mean difference
    of log sampling fractions for the residual computation
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: metagenomeSeq
  role: Provides CSS normalization estimates used as one of the eight compared methods
- name: DESeq2
  role: Provides MED (median) normalization estimates used as one of the eight compared
    methods
- name: edgeR
  role: Provides modified UQ and TMM normalization estimates used as compared methods
- name: dplyr
  role: Required R library dependency for running the ANCOM-BC archive code
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: Required R library dependency for running the ANCOM-BC archive code
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:1716d48d1db82657bbb373f8bf7cf2026ecb54856c484ea7eac54d2cddedd4dc
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# residual-distribution-boxplot-visualization

## Summary

Visualize per-sample sampling-fraction residuals as box plots grouped by normalization method (ANCOM-BC, ELib-UQ, ELib-TMM, CSS, MED, UQ, TMM, TSS), reproducing the paper's Fig. 3 comparison of how well each method removes the bias introduced by differences in sampling fractions.

## When to use

Use when comparing normalization methods on simulated count data with known sampling fractions and you need to show, per sample, the residual between observed and method-estimated log sampling fractions as a grouped box plot.

## When NOT to use

- When the data are raw observed counts with no known (true) sampling fractions — residuals against truth cannot be computed, so only real-data DA testing is appropriate, not this residual comparison
- When sample sizes are very small (e.g. n = 5 per group), where the article notes ANCOM-BC may not perform well and FDR is not controlled

## Inputs

- Simulated microbiome count feature table (taxa x samples) from the unbalanced microbial load / balanced library size scenario generated with the ANCOM-BC Code Archive simulation scripts
- Per-sample group labels for the two experimental groups
- Per-method estimated log-scale sampling fractions (ANCOM-BC output vector d; estimates from CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, TSS)

## Outputs

- method_residual_variances.csv — one residual variance per normalization method
- fig3_residual_boxplot.png — ggplot2-style box plot of sampling-fraction residual distributions across the eight methods with variance annotations

## How to apply

Run each normalization method (ANCOM-BC via ANCOM_BC with tol.EM = 1e-5 and max.iterNum = 100; CSS from metagenomeSeq; MED from DESeq2; modified UQ and TMM from edgeR; plus ELib-UQ, ELib-TMM, TSS) on the same simulated feature table, extracting each method's estimated log-scale (natural log) sampling fractions. Compute per-sample residuals as the deviation between observed and estimated log sampling fractions. Summarize per-method residual variances into a CSV (e.g. method_residual_variances.csv) and draw a ggplot2-style box plot of residual distributions across all eight methods with variance annotations, clustering patterns, and group labels visible (e.g. fig3_residual_boxplot.png). Correct application is judged by ANCOM-BC residuals being intermixed between the two experimental groups without group-label clustering and having the smallest variance, while methods other than UQ and TMM show systematically separated residuals.

## Related tools

- **ANCOM-BC** (Estimates sampling fractions in log scale (output d) and the mean difference of log sampling fractions for the residual computation) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **metagenomeSeq** (Provides CSS normalization estimates used as one of the eight compared methods)
- **DESeq2** (Provides MED (median) normalization estimates used as one of the eight compared methods)
- **edgeR** (Provides modified UQ and TMM normalization estimates used as compared methods)
- **dplyr** (Required R library dependency for running the ANCOM-BC archive code) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (Required R library dependency for running the ANCOM-BC archive code) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Examples

```
ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM = 1e-5, max.iterNum = 100, perNum, alpha)
```

## Evaluation signals

- fig3_residual_boxplot.png exists and contains box plots for all eight normalization methods
- method_residual_variances.csv lists exactly one variance per method, consistent with the plotted residual distributions
- ANCOM-BC residuals show the smallest variance among the eight methods
- ANCOM-BC residuals are intermixed between the two groups without clustering by group labels, while methods other than UQ and TMM show systematically separated residuals in the large sampling-fraction variability scenario

## Limitations

- ANCOM-BC may not perform well with very small sample sizes (n = 5 per group); FDR not controlled in such cases
- ANCOM-BC v1.0 only supports discrete group.var
- Conclusions about residual separation depend on the simulated scenario (large sampling-fraction variability, unbalanced microbial load with balanced library size)

## Evidence

- [discussion] ANCOM-BC is the only method that estimates sampling fractions: "it is the only method available in the literature that estimates the sampling fraction"
- [readme] Estimated sampling fractions in log scale returned by ANCOM_BC: "`d`: A numeric vector. Estimated sampling fractions in log scale (natural log)."
- [methods] E-M convergence parameters for ANCOM_BC: "tol.EM`` Numeric. The iteration convergence tolerance for E-M algorithm."
- [readme] Pre-processing filters applied before normalization comparison: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis."
- [intro] Competing normalization methods from established packages: "we also considered modified versions of UQ and TMM implemented in edgeR11"
