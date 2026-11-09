---
name: microbiome-count-normalization
description: Use when raw OTU/feature count tables need normalization prior to ordination,
  BSS, or differential abundance analysis, and competing normalization methods must
  be ranked on a dataset such as the global gut microbiota data.
when_to_use_negative:
- The input is already normalized relative abundances or the analysis goal does not
  involve comparing normalization methods on count data
- Sample sizes per group are very small (e.g. n = 5 per group), where ANCOM-BC may
  not perform well and FDR is not controlled
- The proportion of differentially abundant taxa is too large, where the method may
  show slightly elevated FDR
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: Bias-correction method whose E-M algorithm estimates sampling fractions and
    produces bias-corrected absolute abundance estimates used as the normalization
    to be ranked
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: metagenomeSeq
  role: Implements the CSS normalization applied to the preprocessed feature table
- name: DESeq2
  role: Implements the Median (MED) normalization applied to the preprocessed feature
    table
- name: edgeR
  role: Implements the modified UQ and TMM normalizations applied to the preprocessed
    feature table
- name: dplyr
  role: R-package dependency required by the ANCOM-BC code for data manipulation
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: R-package dependency required by the ANCOM-BC code for optimization
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:57d1d564ace5c6beac92f802f96b2c172c02eaa2f9f12cb48c8e3f1efbd67b9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - DESeq2
  - R
  - dplyr
  - edgeR
  - metagenomeSeq
  - nloptr
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 22
---

# microbiome-count-normalization

## Summary

Applies and compares microbiome count normalization methods (ANCOM-BC, CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, TSS) on a preprocessed feature table so that cross-sample and cross-group comparisons of absolute abundance are debiased for sampling-fraction differences.

## When to use

Use when raw OTU/feature count tables need normalization prior to ordination, BSS, or differential abundance analysis, and competing normalization methods must be ranked on a dataset such as the global gut microbiota data.

## When NOT to use

- The input is already normalized relative abundances or the analysis goal does not involve comparing normalization methods on count data
- Sample sizes per group are very small (e.g. n = 5 per group), where ANCOM-BC may not perform well and FDR is not controlled
- The proportion of differentially abundant taxa is too large, where the method may show slightly elevated FDR

## Inputs

- Raw OTU/feature count table (taxa in rows, samples in columns) from the global gut microbiota dataset
- Metadata table with sample IDs and a discrete group variable (e.g. nationality: Malawi vs Venezuela)

## Outputs

- Eight normalized abundance tables (ANCOM-BC, CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, TSS)
- NMDS ordination coordinates (first two dimensions) per normalized table
- Ranked BSS table written to bss_ranking.csv with columns (method, BSS, rank)

## How to apply

First preprocess the raw feature table with feature_table_pre_process (e.g. sample.var='sample', zero.cut=0.90 to drop taxa with more than 90% zeroes, lib.cut=1000 to drop low-library-size samples, neg.lb=TRUE to flag structural zeros via the asymptotic lower bound). Then run ANCOM_BC (adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05) whose E-M algorithm estimates the sampling fraction and yields bias-corrected absolute abundances. Apply the competing normalizations to the same preprocessed table: CSS via metagenomeSeq, MED via DESeq2, UQ and TMM via edgeR, plus ELib-UQ, ELib-TMM, and TSS. Finally assess each normalized table by computing NMDS ordination coordinates and the Between-Group Sum of Squares (BSS) among groups, ranking methods by BSS; on the Malawi/Venezuela subset of the global gut microbiota data the expected ordering is ANCOM-BC largest, followed by ELib-TMM and MED.

## Related tools

- **ANCOM-BC** (Bias-correction method whose E-M algorithm estimates sampling fractions and produces bias-corrected absolute abundance estimates used as the normalization to be ranked) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **metagenomeSeq** (Implements the CSS normalization applied to the preprocessed feature table)
- **DESeq2** (Implements the Median (MED) normalization applied to the preprocessed feature table)
- **edgeR** (Implements the modified UQ and TMM normalizations applied to the preprocessed feature table)
- **dplyr** (R-package dependency required by the ANCOM-BC code for data manipulation) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (R-package dependency required by the ANCOM-BC code for optimization) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive

## Examples

```
feature_table_pre_process(feature.table, meta.data, sample.var='sample', group.var='nationality', zero.cut=0.90, lib.cut=1000, neg.lb=TRUE); ANCOM_BC(pre.table, grp.name, grp.ind, struc.zero, adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)
```

## Evaluation signals

- The BSS ranking written to bss_ranking.csv reproduces the reported ordering: ANCOM-BC largest, followed by ELib-TMM and MED, with the remaining methods performing poorly
- feature_table_pre_process removes taxa with proportion of zeroes greater than zero.cut (0.90) and samples with library size less than lib.cut (1000)
- The ANCOM_BC output contains the expected res columns (mean.difference, se, W, p.val, q.val, diff.abn) and the estimated log-scale sampling fraction vector d
- Each of the eight normalized tables yields first two NMDS coordinates on the same preprocessed sample set
- Structural zeros flagged via neg.lb=TRUE are consistent with the structure.zeros matrix returned by preprocessing

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group, and FDR is not controlled in such cases
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var and aims to compare change of absolute abundance across different levels of group.var

## Evidence

- [intro] we also illustrate ANCOM-BC using the well-known global gut microbiota dataset9: "we also illustrate ANCOM-BC using the well-known global gut microbiota dataset9"
- [intro] Median (MED) in DESeq210, Upper Quartile (UQ) and Trimmed Mean of M values (TMM): "Median (MED) in DESeq210, Upper Quartile (UQ) and Trimmed Mean of M values (TMM)"
- [readme] feature_table_pre_process usage signature: "feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb)"
- [readme] Taxa with proportion of zeroes greater than zero.cut are not included in the analysis: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis."
- [readme] Samples with library size less than lib.cut are not included in the analysis: "Samples with library size less than lib.cut are not included in the analysis."
- [discussion] ANCOM-BC is the only method that estimates the sampling fraction: "it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions across samples"
- [intro] Modified UQ and TMM implemented in edgeR: "we also considered modified versions of UQ and TMM implemented in edgeR11"
