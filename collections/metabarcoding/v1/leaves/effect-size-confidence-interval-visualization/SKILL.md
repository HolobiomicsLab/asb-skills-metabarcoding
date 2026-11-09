---
name: effect-size-confidence-interval-visualization
description: Use when ANCOM-BC (or similar DA) output includes mean.difference and
  se per taxon and you need to report effect sizes with CIs, e.g., reproducing Fig.
  6a-style phylum-level plots stratified by infants vs adults.
when_to_use_negative:
- Data are already relative abundances and you only want normalized relative-abundance
  summaries — ANCOM-BC mean differences are on the log absolute-abundance scale after
  sampling-fraction correction, not relative abundances
- Very small sample sizes (e.g., n = 5 per group) — the article notes ANCOM-BC may
  not perform well and FDR is not controlled in such cases
- The goal is a p-value-only summary without effect sizes — use the res table directly
  rather than this visualization step
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: ANCOM-BC
  role: Produces the mean.difference, se, W, p.val, q.val, and diff.abn per taxon
    that the visualization renders
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Implementation language for the plotting and result-export workflow
- name: dplyr
  role: Data manipulation of the ANCOM-BC result table prior to plotting and CSV export
- name: nloptr
  role: Required library dependency for running the archived ANCOM-BC R code
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:23dbd8ab8386b93137a8f2f2fa4b3fe2b2a1a0ac0c1b2127d1d9adee105420d2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# Effect-size confidence-interval visualization

## Summary

Plots ANCOM-BC estimated mean differences of log absolute abundance between experimental groups (e.g., US vs Malawi/Venezuela gut microbiota) with their confidence intervals per taxon, highlighting differentially abundant taxa at q.val < 0.05 so effect sizes are visible alongside significance.

## When to use

Use when ANCOM-BC (or similar DA) output includes mean.difference and se per taxon and you need to report effect sizes with CIs, e.g., reproducing Fig. 6a-style phylum-level plots stratified by infants vs adults.

## When NOT to use

- Data are already relative abundances and you only want normalized relative-abundance summaries — ANCOM-BC mean differences are on the log absolute-abundance scale after sampling-fraction correction, not relative abundances
- Very small sample sizes (e.g., n = 5 per group) — the article notes ANCOM-BC may not perform well and FDR is not controlled in such cases
- The goal is a p-value-only summary without effect sizes — use the res table directly rather than this visualization step

## Inputs

- ANCOM-BC result data frame (res) with per-taxon columns: mean.difference, se, W, p.val, q.val, diff.abn
- Pre-processed OTU/feature table (taxa in rows, samples in columns) and sample metadata with group labels (e.g., nationality, age stratum)

## Outputs

- Forest-style plot of estimated log mean differences of absolute abundance per taxon (phylum) with confidence intervals, differentially abundant taxa (q.val < 0.05) highlighted
- CSV of per-taxon results (taxon, W, p.val, q.val, diff.abn) written via write_csv

## How to apply

Run ANCOM_BC per two-group comparison (adj.method = 'bonferroni', tol.EM = 1e-5, max.iterNum = 100, perNum = 1000, alpha = 0.05) after feature_table_pre_process with zero.cut = 0.90, lib.cut = 1000, neg.lb = TRUE. From the returned res data frame, take mean.difference (natural-log mean difference of absolute abundance between groups) and se to construct two-sided Z-based confidence intervals per taxon. Plot mean differences with their CIs for each phylum, color-highlighting taxa with q.val < 0.05 (diff.abn = TRUE). Validate the figure against the reported findings: rural-associated phyla more abundant in Malawi/Venezuela and Verrucomicrobia significantly increased in US adults and infants. Export the per-taxon table (taxon, W, p.val, q.val, diff.abn) to CSV with write_csv to keep the plot reproducible.

## Related tools

- **ANCOM-BC** (Produces the mean.difference, se, W, p.val, q.val, and diff.abn per taxon that the visualization renders) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Implementation language for the plotting and result-export workflow)
- **dplyr** (Data manipulation of the ANCOM-BC result table prior to plotting and CSV export)
- **nloptr** (Required library dependency for running the archived ANCOM-BC R code)

## Examples

```
out <- ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method = "bonferroni", tol.EM = 1e-5, max.iterNum = 100, perNum = 1000, alpha = 0.05)
```

## Evaluation signals

- Taxa flagged diff.abn = TRUE (bonferroni-adjusted q.val < alpha = 0.05) in the table match the highlighted points in the plot
- CI half-width per taxon equals the expected Z-based multiplier times se computed from the returned mean.difference and se columns
- Reproduced figure confirms reported findings: Verrucomicrobia significantly increased in US relative to Malawi adults/infants and Venezuelan adults, with rural-associated phyla more abundant in Malawi
- Exported CSV contains one row per taxon with non-empty taxon, W, p.val, q.val, diff.abn columns and q.val consistent with bonferroni adjustment of p.val

## Limitations

- ANCOM-BC may not perform well with very small sample sizes (e.g., n = 5 per group); FDR is not controlled in such cases
- Slightly elevated FDR may occur when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var; continuous covariates require the newer ANCOMBC package formulation

## Evidence

- [readme] mean.difference is the natural-log mean difference: "mean.difference``: Numeric. The estimated mean difference of absolute abundance between groups in log scale (natural log)"
- [methods] two-sided Z-test: "P-value obtained from two-sided Z-test using the test statistic W"
- [other] confirms reported finding: [evidence span withheld — anchor status: not_found]
- [abstract] bias correction rationale: "ANCOM-BC), which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples."
- [readme] bonferroni adjustment parameter: "adj.method``: Character. Returns p-values adjusted using the specified method, including "holm", "hochberg", "hommel", "bonferroni", "BH", "BY", "fdr", "none"``"
