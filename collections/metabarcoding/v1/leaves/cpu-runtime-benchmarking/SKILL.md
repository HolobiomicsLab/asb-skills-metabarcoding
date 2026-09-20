---
name: cpu-runtime-benchmarking
description: Use when comparing computational cost of microbiome differential abundance
  methods on synthetic OTU tables with large taxon counts (e.g., 500 or 1000 taxa),
  where runtime differences of minutes vs hours are a decision factor.
when_to_use_negative:
- The goal is statistical performance assessment (FDR control, power, p-values) rather
  than computational cost — use the paper's power/FDR simulation workflow instead
- The number of taxa is small enough that runtime is trivially fast for all methods
  and the benchmark cannot discriminate
- Input is real experimental data (e.g., the dietswap or global gut microbiota datasets)
  — this skill benchmarks on synthetic OTU tables, not empirical DA analysis
tools:
- name: R
  role: Runtime environment in which both methods are executed and system.time CPU
    measurements are taken
- name: dplyr
  role: Required R dependency for running the archived ANCOM-BC timing scripts
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: Required R dependency (optimization) for the ANCOM_BC implementation
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM-BC
  role: Method under benchmark; run via the ANCOM_BC main function with E-M algorithm
    (tol.EM = 1e-5, max.iterNum = 100)
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM
  role: Comparison baseline; permutation-based log-ratio testing with perNum = 1000,
    whose runtime is the expensive reference point
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:c1db527dbe55439ab19c01ff852bea7941d8c3bcc6bfdd0ceae0cf1f29e5f511
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
yes_rate: 0.45454545454545453
n_claims: 11
---

# cpu-runtime-benchmarking

## Summary

Benchmarking methodology for measuring and comparing CPU runtime of differential abundance methods (ANCOM-BC vs ANCOM) on synthetic OTU count tables with large numbers of taxa (500 and 1000). It quantifies the computational cost gap that motivated ANCOM-BC's E-M based design over ANCOM's permutation-based log-ratio testing.

## When to use

Use when comparing computational cost of microbiome differential abundance methods on synthetic OTU tables with large taxon counts (e.g., 500 or 1000 taxa), where runtime differences of minutes vs hours are a decision factor.

## When NOT to use

- The goal is statistical performance assessment (FDR control, power, p-values) rather than computational cost — use the paper's power/FDR simulation workflow instead
- The number of taxa is small enough that runtime is trivially fast for all methods and the benchmark cannot discriminate
- Input is real experimental data (e.g., the dietswap or global gut microbiota datasets) — this skill benchmarks on synthetic OTU tables, not empirical DA analysis

## Inputs

- Synthetic OTU count tables (taxa in rows, samples in columns) generated at 500 and 1000 taxa per the archived timing scripts
- R environment with dplyr and nloptr libraries installed
- Timing scripts from the ANCOM-BC Code Archive

## Outputs

- timing_comparison.csv: table of measured CPU times per method (ANCOM-BC, ANCOM) per taxon count (500, 1000)
- system.time records for each method run

## How to apply

Generate synthetic OTU count tables at the target taxon counts (500 and 1000 taxa) using the timing scripts archived in the ANCOM-BC Code Archive (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive). Run the ANCOM_BC main function with the E-M algorithm parameters from the paper (tol.EM = 1e-5, max.iterNum = 100) and wrap the call in R's system.time to capture CPU time. On the identical datasets, run ANCOM with permutation-based log-ratio testing at perNum = 1000 and record its CPU time the same way. Tabulate measured CPU times per method per taxon count and write the table to timing_comparison.csv. Validate by checking that ANCOM-BC completes in seconds-to-minutes (reported ~0.28 min at 500 taxa and ~0.51 min at 1000 taxa) while ANCOM requires hours (reported ~63 and ~211 min), matching the paper's runtime comparison within expected hardware variation. The rationale: ANCOM's 1000-permutation log-ratio testing scales poorly with taxon count, whereas ANCOM-BC's E-M algorithm recasts DA as linear regression with a sampling-fraction offset, which is computationally simple to implement.

## Related tools

- **R** (Runtime environment in which both methods are executed and system.time CPU measurements are taken)
- **dplyr** (Required R dependency for running the archived ANCOM-BC timing scripts) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (Required R dependency (optimization) for the ANCOM_BC implementation) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM-BC** (Method under benchmark; run via the ANCOM_BC main function with E-M algorithm (tol.EM = 1e-5, max.iterNum = 100)) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM** (Comparison baseline; permutation-based log-ratio testing with perNum = 1000, whose runtime is the expensive reference point)

## Examples

```
system.time(ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="BH", tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05))
```

## Evaluation signals

- ANCOM-BC completes in seconds-to-minutes: reported ~0.28 min at 500 taxa and ~0.51 min at 1000 taxa
- ANCOM requires hours: reported ~63 min at 500 taxa and ~211 min at 1000 taxa
- Both methods are timed on the identical synthetic OTU tables at each taxon count, so the runtime ratio reflects method cost, not input differences
- timing_comparison.csv contains a CPU-time entry for every method x taxon-count combination (4 rows expected)
- Reproduced timings match the paper's reported runtime comparison within expected hardware variation

## Limitations

- Timings are hardware-dependent; only agreement 'within expected hardware variation' with the reported 0.28/63 min (500 taxa) and 0.51/211 min (1000 taxa) values is expected
- ANCOM's perNum argument (set to 1000) dominates its runtime; changing the permutation count invalidates comparison with the reported figures
- ANCOM-BC v1.0 only supports a discrete group.var, so the benchmark scripts are constrained to discrete grouping designs

## Evidence

- [intro] One of the deficiencies of ANCOM is that it does not provide p value for individual taxon, nor can it provide standard errors or confidence intervals of DA for each taxon, and it can be: "One of the deficiencies of ANCOM is that it does not provide p value for individual taxon, nor can it provide standard errors or confidence intervals of DA for each taxon, and it can be"
- [methods] ANCOM_BC function signature with E-M parameters: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [readme] E-M convergence tolerance definition: "tol.EM`` Numeric. The iteration convergence tolerance for E-M algorithm."
- [readme] Permutation count argument definition: "perNum`` Numeric. The maximum number of permutations. This argument is active only if there exist more than 2 groups."
