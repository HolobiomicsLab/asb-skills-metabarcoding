---
name: r-package-workflow-execution
description: Use when reproducing or benchmarking the ANCOM-BC pipeline in R from
  the ANCOM-BC Code Archive — e.g., timing comparisons against ANCOM at large taxon
  counts (500/1000 taxa) on synthetic OTU count tables.
when_to_use_negative:
- The goal is DA analysis itself rather than benchmarking/runtime reproduction — use
  the maintained ANCOMBC Bioconductor package (ancombc/ancombc2) instead of the archived
  v1.0 scripts, which only support discrete group.var
- The dataset is the empirical global gut/dietswap data where the scientific question
  is differential abundance, not CPU-time comparison of methods
tools:
- name: ANCOM-BC
  role: 'Bias-corrected method under benchmark: ANCOM_BC main function run with E-M
    algorithm (tol.EM=1e-5, max.iterNum=100) on synthetic OTU tables while timing
    execution'
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM
  role: 'Baseline comparator: permutation-based log-ratio testing (perNum=1000) run
    on the same synthetic datasets to measure the slower reference runtime'
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Execution environment; scripts wrapped in system.time to record CPU time per
    method per dataset
- name: dplyr
  role: Required R library for the archived code to run
- name: nloptr
  role: Required R library for the archived code to run
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:1c3dcfd454808421ef854bc300f339710e117fa97b51e7dde3dcf993564ccaf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# r-package-workflow-execution

## Summary

Executes the archived ANCOM-BC R workflow (dplyr/nloptr dependencies, feature_table_pre_process, ANCOM_BC main function) to reproduce paper results, including CPU-time benchmarking of ANCOM-BC versus permutation-based ANCOM on synthetic OTU tables with 500 and 1000 taxa.

## When to use

Use when reproducing or benchmarking the ANCOM-BC pipeline in R from the ANCOM-BC Code Archive — e.g., timing comparisons against ANCOM at large taxon counts (500/1000 taxa) on synthetic OTU count tables.

## When NOT to use

- The goal is DA analysis itself rather than benchmarking/runtime reproduction — use the maintained ANCOMBC Bioconductor package (ancombc/ancombc2) instead of the archived v1.0 scripts, which only support discrete group.var
- The dataset is the empirical global gut/dietswap data where the scientific question is differential abundance, not CPU-time comparison of methods

## Inputs

- Synthetic OTU count table (data frame/matrix, taxa in rows, samples in columns) with 500 taxa
- Synthetic OTU count table with 1000 taxa
- Metadata data frame with sample IDs and a discrete group.var
- Timing scripts from github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Outputs

- timing_comparison.csv tabulating CPU time per method (ANCOM-BC, ANCOM) per taxon count (500, 1000)
- ANCOM_BC result data frame (mean.difference, se, W, p.val, q.val, diff.abn, d, mu, bias.em, bias.wls)
- Pre-processed feature table, library sizes, group assignments, and structural-zero matrix from feature_table_pre_process

## How to apply

Install R with the declared dependencies (library(dplyr), library(nloptr)) and pull the timing scripts from the ANCOM-BC Code Archive. Pre-process each synthetic OTU table with feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb), then run the main function ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha) with the paper's E-M settings (tol.EM = 1e-5, max.iterNum = 100), wrapping calls in system.time to record CPU time. Run the permutation-based ANCOM (perNum = 1000) on the identical OTU tables for comparison. Tabulate measured CPU minutes per method per taxon count into timing_comparison.csv and compare against the reported runtimes (~0.28 vs ~63 min at 500 taxa; ~0.51 vs ~211 min at 1000 taxa), allowing hardware variation.

## Related tools

- **ANCOM-BC** (Bias-corrected method under benchmark: ANCOM_BC main function run with E-M algorithm (tol.EM=1e-5, max.iterNum=100) on synthetic OTU tables while timing execution) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM** (Baseline comparator: permutation-based log-ratio testing (perNum=1000) run on the same synthetic datasets to measure the slower reference runtime) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Execution environment; scripts wrapped in system.time to record CPU time per method per dataset)
- **dplyr** (Required R library for the archived code to run)
- **nloptr** (Required R library for the archived code to run)

## Examples

```
Rscript -e 'ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="BH", tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)'
```

## Evaluation signals

- ANCOM-BC completes in seconds-to-minutes (~0.28 min at 500 taxa, ~0.51 min at 1000 taxa) while ANCOM requires hours (~63 and ~211 min), matching the paper within expected hardware variation
- ANCOM_BC call uses the paper's E-M settings: tol.EM = 1e-5 and max.iterNum = 100
- ANCOM baseline uses the specified permutation count perNum = 1000
- timing_comparison.csv contains one measured CPU-time row per method per taxon count (500 and 1000 taxa) with no missing cells
- Pre-processing outputs (library.size, group.name, group.ind, structure.zeros) are returned by feature_table_pre_process before any ANCOM_BC call

## Limitations

- ANCOM-BC v1.0 archived code only supports discrete group.var, so the timing workflow cannot use continuous grouping variables
- Runtime absolutes vary with hardware; validation must tolerate hardware-dependent variation around the reported 0.28/0.51 vs 63/211 minute figures
- ANCOM's permutation-based testing (perNum = 1000) dominates its runtime, so benchmark conclusions are specific to this parameterization

## Evidence

- [methods] R-package dependencies: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [discussion] ANCOM-BC runtime: [evidence span withheld — anchor status: not_found]
- [readme] required libraries: "The following libraries need to be included for the R code to run"
- [other] benchmark finding: [evidence span withheld — anchor status: not_found]
