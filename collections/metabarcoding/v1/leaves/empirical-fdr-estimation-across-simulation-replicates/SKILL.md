---
name: empirical-fdr-estimation-across-simulation-replicates
description: Use when benchmarking differential abundance methods on simulated compositional
  data where ground-truth DA taxa are known, to check which methods control FDR at
  a nominal level (e.g., 5%) while retaining power.
when_to_use_negative:
- The data are real observational data with no known ground truth of DA taxa — empirical
  FDR/power cannot be computed without simulation
- Sample sizes are very small (e.g., n = 5 per group), where the paper notes ANCOM-BC
  may not control FDR
- The proportion of differentially abundant taxa is very large, which can yield slightly
  elevated FDR even for well-controlled methods
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: ANCOM-BC
  role: Primary DA method under evaluation; run via ANCOM_BC with tol.EM=1e-5, max.iterNum=100,
    bonferroni adjustment, alpha=0.05 on each replicate
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM
  role: Competing DA method applied to each replicate under identical significance
    criteria
- name: metagenomeSeq
  role: Competing DA method (ZIG model) applied to each replicate; found up to 70%
    inflated FDR
- name: DESeq2
  role: Competing DA method using MED, UQ, and TMM normalization variants applied
    to each replicate
- name: edgeR
  role: Competing DA method using UQ and TMM variants applied to each replicate; FDR
    reached 40% as sample size grew
- name: R
  role: Execution environment for the simulation scripts and DA method calls
- name: dplyr
  role: R library required by the ANCOM-BC code archive scripts for data manipulation
- name: nloptr
  role: R library required by the ANCOM-BC code archive scripts for optimization in
    the E-M algorithm
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:2319fc49ce647a2e39c4f05afbb8a08b61f1f37bd92a7e9157743a7e23c52a23
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM
  - ANCOM-BC
  - DESeq2
  - R
  - dplyr
  - edgeR
  - metagenomeSeq
  - nloptr
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# empirical-fdr-estimation-across-simulation-replicates

## Summary

Estimate empirical False Discovery Rate and statistical power for differential abundance methods by running each method on many simulated microbiome datasets (e.g., 100 Poisson-Gamma replicates) under known ground truth and comparing declared DA taxa to truly DA taxa.

## When to use

Use when benchmarking differential abundance methods on simulated compositional data where ground-truth DA taxa are known, to check which methods control FDR at a nominal level (e.g., 5%) while retaining power.

## When NOT to use

- The data are real observational data with no known ground truth of DA taxa — empirical FDR/power cannot be computed without simulation
- Sample sizes are very small (e.g., n = 5 per group), where the paper notes ANCOM-BC may not control FDR
- The proportion of differentially abundant taxa is very large, which can yield slightly elevated FDR even for well-controlled methods

## Inputs

- Simulated OTU/feature count tables (taxa in rows, samples in columns) generated under the Poisson-Gamma model, 100 replicates per design
- Ground-truth labels of truly differentially abundant taxa per replicate
- Group metadata (sample IDs and group assignment for unbalanced n1=20, n2=30 and balanced n1=n2=50 designs)
- R environment with dplyr and nloptr libraries and scripts from github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Outputs

- Per-method empirical FDR and power values for each simulation design
- A combined summary table written to fdr_power_table.csv
- Fig. 4-style FDR vs power comparison across DA methods

## How to apply

Generate a fixed number of simulated datasets (the ANCOM-BC paper uses 100 replicates) under a known generative model, here the Poisson-Gamma model, for each design — small unbalanced (n1=20, n2=30) and large balanced (n1=n2=50). Run each competing method (ANCOM-BC, ANCOM, metagenomeSeq ZIG, DESeq2 MED/UQ/TMM, edgeR UQ/TMM) on every replicate with identical significance criteria, e.g., ANCOM-BC via ANCOM_BC with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, alpha=0.05, calling taxa DA when q.val < alpha. For each method and design, compute the empirical false discovery rate (false DA calls divided by total DA calls, averaged over replicates) and power (proportion of truly DA taxa recovered). Assemble per-method FDR and power into a single summary table (fdr_power_table.csv) and compare against the nominal 5% level.

## Related tools

- **ANCOM-BC** (Primary DA method under evaluation; run via ANCOM_BC with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, alpha=0.05 on each replicate) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM** (Competing DA method applied to each replicate under identical significance criteria)
- **metagenomeSeq** (Competing DA method (ZIG model) applied to each replicate; found up to 70% inflated FDR)
- **DESeq2** (Competing DA method using MED, UQ, and TMM normalization variants applied to each replicate)
- **edgeR** (Competing DA method using UQ and TMM variants applied to each replicate; FDR reached 40% as sample size grew)
- **R** (Execution environment for the simulation scripts and DA method calls)
- **dplyr** (R library required by the ANCOM-BC code archive scripts for data manipulation)
- **nloptr** (R library required by the ANCOM-BC code archive scripts for optimization in the E-M algorithm)

## Examples

```
library(dplyr); library(nloptr); res <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="bonferroni", tol.EM=1e-5, max.iterNum=100, alpha=0.05); da <- res$res$diff.abn
```

## Evaluation signals

- In the saved fdr_power_table.csv, only ANCOM and ANCOM-BC show empirical FDR at or below 0.05 in both designs (small unbalanced n1=20/n2=30 and large balanced n1=n2=50)
- All competing methods exceed empirical FDR of 0.05, with edgeR reaching ~40% and metagenomeSeq (ZIG) up to ~70% as sample size grows
- Power for ANCOM and ANCOM-BC is comparable with other methods, so FDR control is not achieved by losing all detections
- FDR estimates are stable as averages over 100 Poisson-Gamma simulation replicates per design
- Each method is called DA taxa under identical criteria (q.val < alpha with alpha = 0.05) so the comparison is fair

## Limitations

- The ANCOM-BC methodology may not perform well when sample sizes are very small, such as n = 5 per group, where FDR is not controlled
- Slightly elevated FDR may occur when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports a discrete group.var, restricting simulation designs to discrete group comparisons

## Evidence

- [results] Only ANCOM and ANCOM-BC control the FDR under the nominal level (5%) while maintaining power comparable with other methods: [evidence span withheld — anchor status: not_found]
- [readme] ANCOM_BC usage with parameters: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [readme] Required R libraries: [evidence span withheld — anchor status: not_found]
- [intro] edgeR and metagenomeSeq FDR inflation: "Although metagenomeSeq8 was specifically developed for microbiome data, it too is subject to inflated FDR under the Gaussian mixture model"
- [abstract] FDR control claim for ANCOM-BC: "it (a) provides statistically valid test with appropriate p-values, (b) provides confidence intervals for differential abundance of each taxon, (c) controls the False Discovery Rate (FDR)"
