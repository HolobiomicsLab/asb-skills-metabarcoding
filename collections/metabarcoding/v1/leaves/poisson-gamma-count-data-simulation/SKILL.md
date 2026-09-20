---
name: poisson-gamma-count-data-simulation
description: Use when benchmarking differential abundance methods for microbiome count
  data under known ground truth, needing simulated absolute/relative abundance datasets
  with controlled FDR and power estimates.
when_to_use_negative:
- Real observational microbiome data without known true DA status — empirical FDR
  and power cannot be computed from ground truth in that setting
- Very small sample sizes such as n=5 per group — the article notes ANCOM-BC may not
  perform well and FDR is not controlled there
- Studies where the proportion of differentially abundant taxa is too large — ANCOM-BC
  may exhibit slightly elevated FDR in that regime
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: Primary DA method under evaluation, run via ANCOM_BC() on each simulated dataset
    with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, alpha=0.05
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM
  role: Competing DA method benchmarked on the same simulated replicates; one of only
    two methods controlling FDR at/below 5%
- name: metagenomeSeq
  role: Competing ZIG DA method applied to each replicate; exhibits FDR inflation
    up to ~70%
- name: DESeq2
  role: Competing DA method applied in MED, UQ, and TMM normalization variants under
    identical significance criteria
- name: edgeR
  role: Competing DA method applied in UQ and TMM variants; reaches ~40% FDR as sample
    size grows
- name: R
  role: Runtime environment for the simulation scripts and all DA method calls
- name: dplyr
  role: Required library for the ANCOM-BC Code Archive scripts to run
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: Required library for the ANCOM-BC optimization (E-M algorithm) in the archive
    scripts
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:adbba80e446986a438ab67f00a280b3856466632b0b410403faf6de2994aa763
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
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# poisson-gamma-count-data-simulation

## Summary

Generate 100 simulated microbiome count datasets under a Poisson-Gamma model for two designs — small unbalanced (n1=20, n2=30) and large balanced (n1=n2=50) — to benchmark differential abundance methods on empirical FDR and power at the 5% nominal level, as in Fig. 4 of the ANCOM-BC paper.

## When to use

Use when benchmarking differential abundance methods for microbiome count data under known ground truth, needing simulated absolute/relative abundance datasets with controlled FDR and power estimates.

## When NOT to use

- Real observational microbiome data without known true DA status — empirical FDR and power cannot be computed from ground truth in that setting
- Very small sample sizes such as n=5 per group — the article notes ANCOM-BC may not perform well and FDR is not controlled there
- Studies where the proportion of differentially abundant taxa is too large — ANCOM-BC may exhibit slightly elevated FDR in that regime

## Inputs

- ANCOM-BC Code Archive Poisson-Gamma simulation R scripts (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive)
- Simulation design parameters: 100 replicates; small unbalanced design n1=20, n2=30; large balanced design n1=n2=50
- R environment with dplyr and nloptr libraries attached

## Outputs

- fdr_power_table.csv with per-method empirical FDR and power for each design
- Per-method DA calls (taxa with q.val < 0.05) for each of the 100 simulated replicates per design
- Empirical FDR/power summaries distinguishing ANCOM and ANCOM-BC (FDR <= 5%) from inflated competitors (edgeR ~40%, metagenomeSeq ZIG up to ~70%)

## How to apply

Load the Poisson-Gamma simulation scripts from the ANCOM-BC Code Archive (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive) into R with dplyr and nloptr attached. Generate 100 replicate datasets per design: small unbalanced (n1=20, n2=30) and large balanced (n1=n2=50), so that true DA taxa are known by construction. Run ANCOM-BC on every replicate via the ANCOM_BC function with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, and alpha=0.05, declaring a taxon differentially abundant when q.val < alpha; apply competing methods (ANCOM, metagenomeSeq ZIG, DESeq2 MED/UQ/TMM, edgeR UQ/TMM) under identical criteria. Compute empirical false discovery rate and power per method per design across the 100 simulations, assemble the summaries into a single table (fdr_power_table.csv), and validate that only ANCOM and ANCOM-BC hold empirical FDR at or below 0.05 in both designs. The rationale is that simulation with known DA status allows direct estimation of FDR inflation: competing methods' FDR rises with sample size (edgeR up to ~40%, metagenomeSeq up to ~70%), while ANCOM-BC's sampling-fraction offset bias correction controls it.

## Related tools

- **ANCOM-BC** (Primary DA method under evaluation, run via ANCOM_BC() on each simulated dataset with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, alpha=0.05) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM** (Competing DA method benchmarked on the same simulated replicates; one of only two methods controlling FDR at/below 5%)
- **metagenomeSeq** (Competing ZIG DA method applied to each replicate; exhibits FDR inflation up to ~70%)
- **DESeq2** (Competing DA method applied in MED, UQ, and TMM normalization variants under identical significance criteria)
- **edgeR** (Competing DA method applied in UQ and TMM variants; reaches ~40% FDR as sample size grows)
- **R** (Runtime environment for the simulation scripts and all DA method calls)
- **dplyr** (Required library for the ANCOM-BC Code Archive scripts to run) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (Required library for the ANCOM-BC optimization (E-M algorithm) in the archive scripts) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Examples

```
library(dplyr); library(nloptr); res <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="bonferroni", tol.EM=1e-5, max.iterNum=100, alpha=0.05)
```

## Evaluation signals

- In both designs, only ANCOM and ANCOM-BC exhibit empirical FDR at or below the 5% nominal level (0.05); all other methods exceed 0.05
- Competing methods show FDR inflation that grows with sample size: edgeR reaches ~40% and metagenomeSeq (ZIG) up to ~70%
- Each design contains 100 simulated replicate datasets (n1=20, n2=30 and n1=n2=50) generated under the Poisson-Gamma model
- fdr_power_table.csv is produced containing per-method FDR and power summaries for both designs
- ANCOM-BC calls use q.val < alpha (0.05) with bonferroni-adjusted p-values from ANCOM_BC(tol.EM=1e-5, max.iterNum=100)

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n=5 per group; FDR is not controlled in such cases
- ANCOM-BC may have slightly elevated FDR when the proportion of differentially abundant taxa is too large
- FDR inflation in competing methods is design-dependent: it increases as sample size grows, so small-design results do not generalize to large designs

## Evidence

- [methods] ANCOM_BC call parameters: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [readme] tol.EM meaning: "tol.EM: Numeric. The iteration convergence tolerance for E-M algorithm."
- [readme] required libraries: "The following libraries need to be included for the R code to run:

r
library(dplyr)
library(nloptr)
"
- [abstract] FDR control claim: "it (a) provides statistically valid test with appropriate p-values, (b) provides confidence intervals for differential abundance of each taxon, (c) controls the False Discovery Rate (FDR)"
- [discussion] small-sample caveat: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
- [abstract] sampling fraction estimation rationale: "ANCOM-BC), which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples."
