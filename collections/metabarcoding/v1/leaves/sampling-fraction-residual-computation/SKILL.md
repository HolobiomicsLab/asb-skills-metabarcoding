---
name: sampling-fraction-residual-computation
description: Use when simulated microbiome count data with known sampling fractions
  are available and you must quantify how well competing normalization methods remove
  bias from differences in sampling fractions across experimental groups.
when_to_use_negative:
- The data are real observed counts with no known ground-truth sampling fractions,
  so residuals cannot be computed
- The goal is differential abundance testing itself rather than evaluating normalization
  quality (use ANCOM-BC's DA workflow instead)
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC performance
  is known to be unreliable
edam_operation: http://edamontology.org/operation_0337
tools:
- name: ANCOM-BC
  role: Estimates sampling fractions in log scale and the mean difference of log sampling
    fractions via E-M; provides the method's own residuals
  repo: https://github.com/FrederickHuangLin/ANCOMBC
- name: metagenomeSeq
  role: Supplies the CSS normalization method applied to the same simulated counts
- name: DESeq2
  role: Supplies the MED (median) normalization method applied to the same simulated
    counts
- name: edgeR
  role: Supplies modified versions of UQ and TMM normalization applied to the same
    simulated counts
- name: dplyr
  role: Required R library dependency for running the archived ANCOM-BC simulation/analysis
    scripts
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: Required R library dependency (numerical optimization) for the archived ANCOM-BC
    code
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:40bbe133cd1daa28fbd42f2116d24134b50f890a1a0f8d5556c7ac99ec688979
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOMBC
  tools:
  - ANCOM-BC
  - DESeq2
  - R
  - dplyr
  - edgeR
  - metagenomeSeq
  - nloptr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# sampling-fraction-residual-computation

## Summary

Reproduces the Fig. 3 comparison of sampling-fraction normalization residuals from the ANCOM-BC paper: compute per-sample residuals between observed and method-estimated log sampling fractions for ANCOM-BC, ELib-UQ, ELib-TMM, CSS, MED, UQ, TMM, and TSS, then summarize variances and plot residual distributions.

## When to use

Use when simulated microbiome count data with known sampling fractions are available and you must quantify how well competing normalization methods remove bias from differences in sampling fractions across experimental groups.

## When NOT to use

- The data are real observed counts with no known ground-truth sampling fractions, so residuals cannot be computed
- The goal is differential abundance testing itself rather than evaluating normalization quality (use ANCOM-BC's DA workflow instead)
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC performance is known to be unreliable

## Inputs

- Simulated OTU/feature count table (taxa in rows, samples in columns) with known true sampling fractions and group labels
- Metadata data frame containing sample IDs and a discrete group.var
- R environment with dplyr, nloptr, metagenomeSeq, DESeq2, edgeR, ggplot2

## Outputs

- Per-sample sampling-fraction residuals for each of the eight normalization methods (log scale, natural log)
- method_residual_variances.csv with one residual variance per method
- fig3_residual_boxplot.png — box plot of residual distributions across methods with variance annotations

## How to apply

Generate simulated feature tables (e.g., the unbalanced microbial load, balanced library size scenario) from the ANCOM-BC Code Archive simulation scripts in R with dplyr and nloptr. Pre-process counts with feature_table_pre_process (zero.cut, lib.cut, neg.lb for structural zeros via asymptotic lower bounds), then run ANCOM_BC with the E-M algorithm (tol.EM = 1e-5, max.iterNum = 100), extracting the estimated sampling fractions in log scale (natural log) and the estimated mean difference of log sampling fractions. Apply competing normalizers to the same counts — CSS from metagenomeSeq, MED from DESeq2, modified UQ and TMM from edgeR, plus ELib-UQ, ELib-TMM, and TSS — and compute each sample's residual as the deviation between observed and method-estimated log sampling fractions. Summarize per-method residual variances and render a ggplot2-style box plot comparing all eight methods; in the large sampling-fraction variability scenario ANCOM-BC residuals should be intermixed between the two groups while other methods (except UQ and TMM) show systematically separated residuals.

## Related tools

- **ANCOM-BC** (Estimates sampling fractions in log scale and the mean difference of log sampling fractions via E-M; provides the method's own residuals) — https://github.com/FrederickHuangLin/ANCOMBC
- **metagenomeSeq** (Supplies the CSS normalization method applied to the same simulated counts)
- **DESeq2** (Supplies the MED (median) normalization method applied to the same simulated counts)
- **edgeR** (Supplies modified versions of UQ and TMM normalization applied to the same simulated counts)
- **dplyr** (Required R library dependency for running the archived ANCOM-BC simulation/analysis scripts) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (Required R library dependency (numerical optimization) for the archived ANCOM-BC code) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Examples

```
pre = feature_table_pre_process(feature.table, meta.data, sample.var = 'sample', group.var = 'group', zero.cut, lib.cut, neg.lb); res = ANCOM_BC(pre$feature.table, pre$group.name, pre$group.ind, pre$structure.zeros, adj.method = 'BH', tol.EM = 1e-5, max.iterNum = 100, perNum, alpha)
```

## Evaluation signals

- fig3_residual_boxplot.png exists and shows box plots for all eight methods (ANCOM-BC, ELib-UQ, ELib-TMM, CSS, MED, UQ, TMM, TSS)
- method_residual_variances.csv contains exactly one variance value per method, consistent with the plotted residual distributions
- In the large sampling-fraction variability scenario, ANCOM-BC residuals are intermixed between the two groups (no clustering by group label) and have the smallest variance in sampling-fraction estimates
- Existing methods other than UQ and TMM show systematically separated residuals between the two groups, matching the paper's Fig. 3 pattern
- ANCOM-BC estimated sampling fractions are on natural-log scale and the estimated mean difference of log sampling fractions (bias.em/bias.wls) is reproducible from the E-M run with tol.EM = 1e-5

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports a discrete group.var for cross-sectional group comparisons

## Evidence

- [discussion] it is the only method available in the literature that estimates the sampling fraction: "it is the only method available in the literature that estimates the sampling fraction"
- [readme] d: A numeric vector. Estimated sampling fractions in log scale (natural log).: "d`` A numeric vector. Estimated sampling fractions in log scale (natural log)."
- [readme] bias.em`` Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm.: "bias.em`` Numeric. Estimated mean difference of log (natural log) sampling fractions between groups through E-M algorithm."
- [methods] TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound: "TRUE indicates a taxon would be classified as a structural zero in the corresponding experimental group using its asymptotic lower bound"
- [intro] we also considered modified versions of UQ and TMM implemented in edgeR: "we also considered modified versions of UQ and TMM implemented in edgeR"
- [discussion] The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group.: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
