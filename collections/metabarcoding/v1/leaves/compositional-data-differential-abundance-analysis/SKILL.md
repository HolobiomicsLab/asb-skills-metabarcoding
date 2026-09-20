---
name: compositional-data-differential-abundance-analysis
description: Use when an observed OTU/feature count table (taxa x samples) plus metadata
  with a discrete grouping variable must be tested for taxa whose absolute abundance
  differs between groups, while correcting for unequal sampling fractions across samples.
when_to_use_negative:
- 'Sample sizes are very small (e.g. n = 5 per group): the article states ANCOM-BC
  may not perform well and FDR is not controlled in such cases'
- 'The proportion of differentially abundant taxa is too large: FDR may be slightly
  elevated'
- 'The grouping variable is continuous or the goal is correlation estimation rather
  than discrete-group DA: ANCOM-BC v1.0 only supports discrete group.var'
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0091
tools:
- name: ANCOM-BC
  role: 'Primary DA method: preprocessing + E-M bias-corrected log-scale linear regression
    with Z-test inference'
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM
  role: Permutation-based log-ratio DA method used as the computational-cost and statistical
    baseline comparator (perNum = 1000 permutations)
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: Required R library dependency for running the archived ANCOM-BC code
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: Required R library dependency for running the archived ANCOM-BC code
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:f200189a9c0f2294439d92d1f938dd83d68f7e6482dc292b0b93b451becabc27
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ANCOM
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# compositional-data-differential-abundance-analysis

## Summary

Performs differential abundance (DA) analysis of microbiome count data with ANCOM-BC, which recasts DA as a log-scale linear regression with a sample-specific offset that estimates and corrects the sampling-fraction bias, yielding p-values, confidence intervals, and FDR control that compositionality-blind or ANCOM-style methods lack.

## When to use

Use when an observed OTU/feature count table (taxa x samples) plus metadata with a discrete grouping variable must be tested for taxa whose absolute abundance differs between groups, while correcting for unequal sampling fractions across samples.

## When NOT to use

- Sample sizes are very small (e.g. n = 5 per group): the article states ANCOM-BC may not perform well and FDR is not controlled in such cases
- The proportion of differentially abundant taxa is too large: FDR may be slightly elevated
- The grouping variable is continuous or the goal is correlation estimation rather than discrete-group DA: ANCOM-BC v1.0 only supports discrete group.var

## Inputs

- Feature table: data frame or matrix of observed OTU counts with OTUs/taxa in rows and samples in columns
- Metadata data frame containing sample IDs (sample.var) and a discrete group variable (group.var), e.g. 'nationality' for two-group comparison
- Preprocessing parameters: zero.cut (fraction 0-1), lib.cut (numeric library-size threshold), neg.lb (logical structural-zero flag)

## Outputs

- Result data frame 'res' per taxon: mean.difference (log-scale, natural log), se, W test statistic, p.val (two-sided Z-test), q.val (adjusted p-value), diff.abn (logical, q.val < alpha)
- Numeric vector 'd': estimated sampling fractions in log (natural log) scale
- Vector 'mu': estimated log mean absolute abundance for each group
- Matrix 'structure.zeros' of 0/1 indicating structural zeros per taxon per group

## How to apply

First run the preprocessing function feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb) to filter taxa by zero proportion (zero.cut), samples by library size (lib.cut), and identify structural zeros via asymptotic lower bounds. Then call the main function ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha): it introduces a sample-specific offset term in a log-scale linear regression framework (analogous to a log-ratio transformation), estimates log-scale sampling fractions d via an E-M algorithm with a chosen convergence tolerance (tol.EM) and iteration cap (max.iterNum), and tests each taxon with a two-sided Z-test on W = mean.difference/se. P-values are adjusted with the selected method (e.g. 'BH') and taxa with q.val < alpha are declared differentially abundant. The rationale is that testing mean relative abundance is not equivalent to testing mean absolute abundance, so bias from differing sampling fractions must be estimated and removed before inference.

## Related tools

- **ANCOM-BC** (Primary DA method: preprocessing + E-M bias-corrected log-scale linear regression with Z-test inference) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM** (Permutation-based log-ratio DA method used as the computational-cost and statistical baseline comparator (perNum = 1000 permutations)) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (Required R library dependency for running the archived ANCOM-BC code) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (Required R library dependency for running the archived ANCOM-BC code) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Examples

```
ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method = "BH", tol.EM = 1e-5, max.iterNum = 100, perNum = 1000, alpha = 0.05)
```

## Evaluation signals

- Schema check: 'res' contains exactly mean.difference, se, W, p.val, q.val, diff.abn columns, one row per preprocessed taxon
- Invariant: W equals mean.difference/se, and diff.abn is TRUE if and only if q.val < alpha
- Structural-zero matrix has the same number of rows as taxa and columns as group levels, entries only 0 or 1
- Runtime sanity check on synthetic data: ANCOM-BC completes in ~0.28 min at 500 taxa and ~0.51 min at 1000 taxa (E-M, tol.EM = 1e-5, max.iterNum = 100), versus ~63 and ~211 min for ANCOM (perNum = 1000)
- q.val values all fall in [0, 1] and match p.val after the chosen adj.method (e.g. 'BH')

## Limitations

- May not perform well with very small sample sizes (n = 5 per group); FDR is not controlled in that regime
- Slightly elevated FDR when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports a discrete group.var; perNum (permutations) is active only when there exist more than 2 groups

## Evidence

- [abstract] sampling fraction definition: "we define the notion of "sampling fraction" and demonstrate a major hurdle in performing DA analysis of microbiome data is the bias introduced by differences in the sampling fractions"
- [intro] ANCOM limitation: [evidence span withheld — anchor status: not_found]
- [methods] Z-test inference: "P-value obtained from two-sided Z-test using the test statistic W"
- [methods] preprocessing filter: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [discussion] uniqueness of method: "it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions"
- [readme] input format: "Data frame or matrix representing observed OTU table with OTUs (or taxa) in rows and samples in columns."
