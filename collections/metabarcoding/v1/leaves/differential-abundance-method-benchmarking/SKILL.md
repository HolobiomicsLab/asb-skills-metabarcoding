---
name: differential-abundance-method-benchmarking
description: Use when evaluating which microbiome DA method (e.g., ANCOM-BC, ANCOM,
  metagenomeSeq, DESeq2, edgeR) controls FDR at a nominal level (e.g., 5%) while maintaining
  power, via simulation across balanced/unbalanced two-group designs.
when_to_use_negative:
- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC itself does
  not control FDR well and benchmark conclusions may not generalize
- The proportion of differentially abundant taxa in the simulation is too large, which
  can cause slightly elevated FDR and confound the comparison
- The analysis goal is a single-dataset DA test rather than a method comparison —
  use the individual method (e.g., ANCOM-BC with two-group comparison) directly instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: ANCOM-BC
  role: Primary DA method under benchmark; run via ANCOM_BC function with tol.EM=1e-5,
    max.iterNum=100, bonferroni adjustment, alpha=0.05
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM
  role: Competing DA method based on Aitchison's methodology using relative abundances;
    compared on FDR and power
- name: metagenomeSeq
  role: Competing DA method (ZIG model) subject to inflated FDR under the Gaussian
    mixture model; compared on FDR and power
- name: DESeq2
  role: Competing DA method applied with Median (MED), Upper Quartile (UQ), and TMM
    normalization variants
- name: edgeR
  role: Competing DA method applied with modified UQ and TMM normalization variants
- name: R
  role: Implementation environment; requires library(dplyr) and library(nloptr) for
    the archived scripts
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: R library dependency required by the archived ANCOM-BC simulation scripts
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: R library dependency required by the archived ANCOM-BC simulation scripts
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:f7cd0451148763799719f6729bb1b7fcc6bd973c755a57770aff1d771b59aa64
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# differential-abundance-method-benchmarking

## Summary

Benchmark differential abundance (DA) methods for microbiome compositional data by simulating data under a Poisson-Gamma model and comparing empirical False Discovery Rate (FDR) and power across designs, as done in the ANCOM-BC Nature Communications paper.

## When to use

Use when evaluating which microbiome DA method (e.g., ANCOM-BC, ANCOM, metagenomeSeq, DESeq2, edgeR) controls FDR at a nominal level (e.g., 5%) while maintaining power, via simulation across balanced/unbalanced two-group designs.

## When NOT to use

- Sample sizes are very small (e.g., n = 5 per group), where ANCOM-BC itself does not control FDR well and benchmark conclusions may not generalize
- The proportion of differentially abundant taxa in the simulation is too large, which can cause slightly elevated FDR and confound the comparison
- The analysis goal is a single-dataset DA test rather than a method comparison — use the individual method (e.g., ANCOM-BC with two-group comparison) directly instead

## Inputs

- Simulated OTU/feature count tables generated under the Poisson-Gamma model (taxa in rows, samples in columns)
- Metadata data frame with sample IDs and a discrete group variable (group.var)
- R scripts from the ANCOM-BC Code Archive (github.com/FrederickHuangLin/ANCOM-BC-Code-Archive)
- R session with dplyr and nloptr libraries attached

## Outputs

- Per-method empirical FDR and power values within each design across 100 simulations
- A combined summary table of FDR and power for all methods (e.g., fdr_power_table.csv)

## How to apply

Generate 100 simulated microbiome datasets under the Poisson-Gamma model for each design — small and unbalanced (n1=20, n2=30) and large and balanced (n1=n2=50). Run ANCOM-BC on every simulated dataset via the ANCOM_BC function with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, and alpha=0.05, calling taxa differentially abundant when q.val < alpha; preprocess tables first with feature_table_pre_process. Apply the competing methods (ANCOM, metagenomeSeq ZIG, DESeq2 MED/UQ/TMM, edgeR UQ/TMM) to each replicate under identical significance criteria, then compute empirical FDR and power per method within each design. The rationale is that methods ignoring sampling-fraction bias show FDR inflation that grows with sample size, so both a small unbalanced and a large balanced design must be compared. Assemble per-method FDR and power summaries into a single table (e.g., fdr_power_table.csv) and verify that only methods correcting for sampling-fraction differences (ANCOM and ANCOM-BC) stay at or below the nominal 5% FDR in both designs.

## Related tools

- **ANCOM-BC** (Primary DA method under benchmark; run via ANCOM_BC function with tol.EM=1e-5, max.iterNum=100, bonferroni adjustment, alpha=0.05) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM** (Competing DA method based on Aitchison's methodology using relative abundances; compared on FDR and power)
- **metagenomeSeq** (Competing DA method (ZIG model) subject to inflated FDR under the Gaussian mixture model; compared on FDR and power)
- **DESeq2** (Competing DA method applied with Median (MED), Upper Quartile (UQ), and TMM normalization variants)
- **edgeR** (Competing DA method applied with modified UQ and TMM normalization variants)
- **R** (Implementation environment; requires library(dplyr) and library(nloptr) for the archived scripts) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (R library dependency required by the archived ANCOM-BC simulation scripts) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (R library dependency required by the archived ANCOM-BC simulation scripts) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Examples

```
Rscript -e 'library(dplyr); library(nloptr); out <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method="bonferroni", tol.EM=1e-5, max.iterNum=100, alpha=0.05)'
```

## Evaluation signals

- In the saved fdr_power_table.csv, only ANCOM and ANCOM-BC exhibit empirical FDR at or below the 5% nominal level (≤ 0.05) in both designs
- All competing methods (metagenomeSeq ZIG, DESeq2 MED/UQ/TMM, edgeR UQ/TMM) exceed FDR = 0.05, with FDR increasing as sample size grows (edgeR reaching ~40% and metagenomeSeq ZIG up to ~70%)
- ANCOM-BC maintains power comparable with other methods while controlling FDR
- Each method is evaluated across exactly 100 simulations per design under identical significance criteria (q.val < 0.05)

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group, where FDR is not controlled
- Slightly elevated FDR may occur when the proportion of differentially abundant taxa is too large
- ANCOM does not provide p-values, standard errors, or confidence intervals per taxon and is computationally intensive

## Evidence

- [results] FDR control finding: [evidence span withheld — anchor status: not_found]
- [results] FDR inflation of competitors: [evidence span withheld — anchor status: not_found]
- [intro] metagenomeSeq FDR inflation: "Although metagenomeSeq8 was specifically developed for microbiome data, it too is subject to inflated FDR under the Gaussian mixture model"
- [readme] ANCOM-BC function signature: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [readme] R dependencies: "library(dplyr)
library(nloptr)"
- [intro] ANCOM limitations: "One of the deficiencies of ANCOM is that it does not provide p value for individual taxon, nor can it provide standard errors or confidence intervals of DA for each taxon"
- [discussion] Small-sample limitation: "The ANCOM-BC methodology may not perform well when the sample sizes are very small, such as n = 5 per group."
- [abstract] FDR control claim: "it (a) provides statistically valid test with appropriate p-values, (b) provides confidence intervals for differential abundance of each taxon, (c) controls the False Discovery Rate (FDR)"
