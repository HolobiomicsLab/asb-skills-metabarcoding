---
name: microbiome-compositional-data-analysis
description: Use when evaluating which DA method controls FDR at a nominal level (e.g.
  5%) while maintaining power on compositional microbiome count data, or when reproducing
  simulation-based method comparisons across balanced and unbalanced group designs.
when_to_use_negative:
- Input is an already-analyzed real dataset rather than simulated data — this skill
  is a simulation benchmark for method comparison, not a DA analysis of observed data
- Sample sizes are very small (e.g. n = 5 per group), where ANCOM-BC may not control
  FDR
- The proportion of differentially abundant taxa is very large, which can slightly
  elevate FDR
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: Primary DA method under evaluation; run via ANCOM_BC with tol.EM=1e-5, max.iterNum=100,
    bonferroni adjustment, alpha=0.05
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM
  role: Competing DA method applied to each replicate under identical significance
    criteria (q.val < alpha)
- name: metagenomeSeq
  role: Competing ZIG DA method whose FDR inflation is measured in the benchmark
- name: DESeq2
  role: Competing DA method applied with MED, UQ, and TMM normalization variants
- name: edgeR
  role: Competing DA method applied with UQ and TMM normalization variants
- name: R
  role: Execution environment for the simulation and DA pipeline
- name: dplyr
  role: Required R library for the ANCOM-BC Code Archive scripts
- name: nloptr
  role: Required R library for the ANCOM-BC Code Archive scripts
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:b33ce001e92ba8ddf89e57439a0589f1430f93aa84a5ecd5f6365e383aaf126c
schema_version: 0.2.0
metadata:
  license_tier: open
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# microbiome-compositional-data-analysis

## Summary

Benchmark differential abundance (DA) methods for compositional microbiome data by simulating data under a Poisson-Gamma model, running ANCOM-BC (and competitors ANCOM, metagenomeSeq, DESeq2, edgeR) on each replicate, and comparing empirical FDR and power across two-group designs.

## When to use

Use when evaluating which DA method controls FDR at a nominal level (e.g. 5%) while maintaining power on compositional microbiome count data, or when reproducing simulation-based method comparisons across balanced and unbalanced group designs.

## When NOT to use

- Input is an already-analyzed real dataset rather than simulated data — this skill is a simulation benchmark for method comparison, not a DA analysis of observed data
- Sample sizes are very small (e.g. n = 5 per group), where ANCOM-BC may not control FDR
- The proportion of differentially abundant taxa is very large, which can slightly elevate FDR

## Inputs

- Simulated OTU/feature count tables (taxa in rows, samples in columns) generated under the Poisson-Gamma model
- Metadata data frame with sample IDs and discrete group labels (group.var)
- ANCOM-BC Code Archive simulation and analysis scripts (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive)
- R session with dplyr and nloptr libraries attached

## Outputs

- fdr_power_table.csv containing per-method empirical FDR and power summaries for each design (n1=20,n2=30 and n1=n2=50) across 100 simulations
- Per-replicate DA calls (q.val < alpha) from ANCOM-BC, ANCOM, metagenomeSeq, DESeq2 (MED/UQ/TMM), and edgeR (UQ/TMM)

## How to apply

Generate 100 simulated microbiome datasets under the Poisson-Gamma model for each design — small and unbalanced (n1=20, n2=30) and large and balanced (n1=n2=50) — using the scripts from the ANCOM-BC Code Archive with the dplyr and nloptr libraries attached in R. Run ANCOM-BC on every replicate via the ANCOM_BC function with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, and alpha=0.05, calling taxa differentially abundant when q.val < alpha; apply the same significance criteria to ANCOM, metagenomeSeq (ZIG), DESeq2 MED/UQ/TMM, and edgeR UQ/TMM variants. Compute the empirical false discovery rate and power per method per design, assemble the per-method summaries into a single table, and write it to fdr_power_table.csv. The rationale is that sampling-fraction differences bias standard methods on compositional data, so the benchmark reveals which methods control FDR as sample size grows.

## Related tools

- **ANCOM-BC** (Primary DA method under evaluation; run via ANCOM_BC with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, alpha=0.05) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM** (Competing DA method applied to each replicate under identical significance criteria (q.val < alpha))
- **metagenomeSeq** (Competing ZIG DA method whose FDR inflation is measured in the benchmark)
- **DESeq2** (Competing DA method applied with MED, UQ, and TMM normalization variants)
- **edgeR** (Competing DA method applied with UQ and TMM normalization variants)
- **R** (Execution environment for the simulation and DA pipeline)
- **dplyr** (Required R library for the ANCOM-BC Code Archive scripts)
- **nloptr** (Required R library for the ANCOM-BC Code Archive scripts)

## Examples

```
library(dplyr); library(nloptr); res <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="bonferroni", tol.EM=1e-5, max.iterNum=100, alpha=0.05)
```

## Evaluation signals

- In the saved fdr_power_table.csv, only ANCOM and ANCOM-BC exhibit empirical FDR at or below the 5% nominal level in both designs (n1=20,n2=30 and n1=n2=50)
- All non-ANCOM/ANCOM-BC methods exceed empirical FDR of 0.05, with FDR increasing as sample size grows (edgeR reaching ~40%, metagenomeSeq ZIG up to ~70%)
- ANCOM and ANCOM-BC maintain power comparable with other methods despite FDR control
- Exactly 100 simulated replicates per design were generated and analyzed before aggregating FDR and power
- ANCOM-BC results were produced with tol.EM=1e-5, max.iterNum=100, bonferroni-adjusted q.val, and taxa called DA when q.val < 0.05

## Limitations

- ANCOM-BC methodology may not perform well when sample sizes are very small, such as n = 5 per group; FDR is not controlled in such cases
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports a discrete group.var and does not handle continuous main variables

## Evidence

- [results] Only ANCOM and ANCOM-BC control FDR under nominal level (5%) while maintaining power comparable with other methods: [evidence span withheld — anchor status: not_found]
- [results] Other methods' FDR increases with sample size: [evidence span withheld — anchor status: not_found]
- [readme] ANCOM_BC function signature: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [methods] R dependencies: "The following libraries need to be included for the R code to run"
- [readme] Significance criterion: "diff.abn. Logical. TRUE if the taxon has q.val less than alpha"
- [discussion] Small-sample caveat: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
