---
name: microbiome-simulation-data-generation
description: Use when a study needs controlled synthetic OTU/count data — e.g., known
  sampling fractions, 500–1000 taxa, unbalanced microbial load — to test bias correction,
  normalization residuals, or CPU-time scaling of DA methods.
when_to_use_negative:
- The input is already an observed feature table from a real experiment — use feature_table_pre_process
  + ANCOM_BC directly rather than simulating data
- The goal is DA inference on real data with confidence intervals and FDR control,
  not method benchmarking — simulation is unnecessary overhead
- The study design requires very small sample sizes (e.g., n = 5 per group), where
  the article notes ANCOM-BC does not control FDR, making benchmark conclusions unreliable
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3697
tools:
- name: R
  role: Runtime environment in which the archived simulation and ANCOM-BC scripts
    are executed
- name: dplyr
  role: Required R dependency for the archived simulation and ANCOM-BC scripts
- name: nloptr
  role: Required R dependency used by ANCOM_BC's optimization routines
- name: ANCOM-BC
  role: Primary method applied to simulated counts; supplies estimated sampling fractions
    (d) and bias estimates (bias.em, bias.wls) used as ground-truth comparisons
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: metagenomeSeq
  role: Provides CSS normalization applied to the same simulated counts for residual
    comparison
- name: DESeq2
  role: Provides the MED normalization applied to the simulated counts
- name: edgeR
  role: Provides modified UQ and TMM normalizations applied to the simulated counts
- name: ANCOM
  role: Permutation-based log-ratio DA method run on the same synthetic tables (perNum
    = 1000) for CPU-timing comparison
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:b70f8c5359a4d58e2433335d5c82b9e78020bde99075755f2b98f785f7bc2a1b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# microbiome-simulation-data-generation

## Summary

Generates synthetic microbiome OTU count tables (e.g., 500 or 1000 taxa, unbalanced microbial load with balanced library size) using the simulation scripts archived in the ANCOM-BC-Code-Archive, in R with dplyr and nloptr, to benchmark normalization and differential-abundance methods such as ANCOM-BC against CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, and TSS.

## When to use

Use when a study needs controlled synthetic OTU/count data — e.g., known sampling fractions, 500–1000 taxa, unbalanced microbial load — to test bias correction, normalization residuals, or CPU-time scaling of DA methods.

## When NOT to use

- The input is already an observed feature table from a real experiment — use feature_table_pre_process + ANCOM_BC directly rather than simulating data
- The goal is DA inference on real data with confidence intervals and FDR control, not method benchmarking — simulation is unnecessary overhead
- The study design requires very small sample sizes (e.g., n = 5 per group), where the article notes ANCOM-BC does not control FDR, making benchmark conclusions unreliable

## Inputs

- R simulation scripts from the ANCOM-BC-Code-Archive (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive)
- Scenario specification: number of taxa (e.g., 500 or 1000), sampling-fraction variability scenario (e.g., unbalanced microbial load with balanced library size)
- R environment with library(dplyr) and library(nloptr)
- Analysis parameters: zero.cut, lib.cut, neg.lb, tol.EM = 1e-5, max.iterNum = 100, perNum = 1000 (for ANCOM permutations)

## Outputs

- Synthetic OTU count tables (taxa in rows, samples in columns) with known true sampling fractions
- Estimated log-scale (natural log) sampling fractions (ANCOM_BC output d) and mean difference of log sampling fractions (bias.em, bias.wls)
- Per-sample sampling-fraction residuals for each of the eight normalization methods, summarized in method_residual_variances.csv
- Fig. 3 residual box plot (fig3_residual_boxplot.png) and CPU timing table (timing_comparison.csv)

## How to apply

Source the simulation scripts from the ANCOM-BC Code Archive (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive) and run them in R with the dplyr and nloptr libraries loaded. Generate count tables under the intended scenario (e.g., unbalanced microbial load with balanced library size, large sampling-fraction variability; or 500-taxa and 1000-taxa tables for timing benchmarks). Pre-process generated tables with feature_table_pre_process, removing taxa with zero proportion above zero.cut, samples with library size below lib.cut, and flagging structural zeros via neg.lb using asymptotic lower bounds. Analyze the simulated data with ANCOM_BC (E-M algorithm, tol.EM = 1e-5, max.iterNum = 100), extracting the estimated log-scale sampling fractions (output vector d) and the estimated mean difference of log sampling fractions (bias.em/bias.wls); apply competing normalizations (CSS via metagenomeSeq, MED via DESeq2, modified UQ and TMM via edgeR, plus ELib-UQ, ELib-TMM, TSS) to the same counts, and record CPU times with system.time for benchmarks. Rationale: simulations with known true sampling fractions allow residuals between estimated and true sampling fractions — the paper's Fig. 3 metric — to be computed exactly, which is impossible on real data.

## Related tools

- **R** (Runtime environment in which the archived simulation and ANCOM-BC scripts are executed)
- **dplyr** (Required R dependency for the archived simulation and ANCOM-BC scripts)
- **nloptr** (Required R dependency used by ANCOM_BC's optimization routines)
- **ANCOM-BC** (Primary method applied to simulated counts; supplies estimated sampling fractions (d) and bias estimates (bias.em, bias.wls) used as ground-truth comparisons) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **metagenomeSeq** (Provides CSS normalization applied to the same simulated counts for residual comparison)
- **DESeq2** (Provides the MED normalization applied to the simulated counts)
- **edgeR** (Provides modified UQ and TMM normalizations applied to the simulated counts)
- **ANCOM** (Permutation-based log-ratio DA method run on the same synthetic tables (perNum = 1000) for CPU-timing comparison)

## Examples

```
library(dplyr); library(nloptr); ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method = "BH", tol.EM = 1e-5, max.iterNum = 100)
```

## Evaluation signals

- Verify the simulation run reproduces the reported timing gap: ANCOM-BC ~0.28 min at 500 taxa and ~0.51 min at 1000 taxa vs ANCOM ~63 min and ~211 min (within hardware variation)
- Check that in the large sampling-fraction-variability scenario, ANCOM-BC residuals are intermixed between the two groups (no clustering by group label) and have the smallest variance among the eight methods
- Confirm method_residual_variances.csv lists exactly one variance per method (ANCOM-BC, ELib-UQ, ELib-TMM, CSS, MED, UQ, TMM, TSS) consistent with the distributions in fig3_residual_boxplot.png
- Confirm fig3_residual_boxplot.png contains box plots for all eight methods with variance annotations
- Check that pre-processing respected zero.cut, lib.cut, and neg.lb (structural zeros flagged via asymptotic lower bounds) before running ANCOM_BC

## Limitations

- ANCOM-BC (and hence simulation-based benchmarking of it) may not perform well at very small sample sizes, such as n = 5 per group, where FDR is not controlled
- Slightly elevated FDR can occur when the proportion of differentially abundant taxa in the simulation is too large (Supplementary Fig. 7a, b)
- The archived ANCOM-BC v1.0 code only supports a discrete group.var, so simulations must use discrete experimental groups

## Evidence

- [readme] R dependencies for archived scripts: "The following libraries need to be included for the R code to run:

library(dplyr)
library(nloptr)"
- [readme] Feature table input format: "feature.table`` Data frame or matrix representing observed OTU table with OTUs (or taxa) in rows and samples in columns."
- [methods] Pre-processing filters: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [methods] E-M parameters: "The iteration convergence tolerance for E-M algorithm"
- [readme] Main function signature returning sampling fractions: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [discussion] Small-sample limitation: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
