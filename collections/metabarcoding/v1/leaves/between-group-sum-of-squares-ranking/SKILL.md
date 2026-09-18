---
name: between-group-sum-of-squares-ranking
description: Use when decision support for choosing a normalization method when you
  have a preprocessed feature table with two known sample groups and need an objective,
  visualization-linked ranking of competing normalizations via NMDS separation.
when_to_use_negative:
- When the question is differential abundance testing with p-values rather than ranking
  normalization methods for between-group separation
- When samples cannot be assigned to discrete groups — ANCOM-BC v1.0 only supports
  discrete group.var and BSS here is computed between group levels
- When the goal is multi-group analysis rather than a two-group (Malawi/Venezuela-style)
  comparison; perNum permutations only activate for more than 2 groups and the reported
  ranking is for two groups
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: ANCOM-BC
  role: Provides the bias-corrected absolute abundance estimates (via E-M estimation
    of sampling fractions) that are ranked against competing normalizations
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: metagenomeSeq
  role: Supplies the CSS normalization whose NMDS/BSS performance is compared
- name: DESeq2
  role: Supplies the MED (median) normalization whose NMDS/BSS performance is compared
- name: edgeR
  role: Supplies the UQ and TMM normalizations whose NMDS/BSS performance is compared
- name: dplyr
  role: R data-manipulation dependency for the ANCOM-BC scripts used in the workflow
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: R optimization dependency required by the ANCOM-BC code
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: R
  role: Implementation environment for preprocessing, normalization, NMDS, and BSS
    computation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:6c4ea3808b9e3730c48f8e5153126741023e3b3d6661d793b5c2101d69c03917
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# between-group-sum-of-squares-ranking

## Summary

Ranks normalization methods for a two-group microbiome comparison by computing Between-Group Sum of Squares (BSS) on the first two NMDS ordination coordinates of each normalized feature table, as done in the ANCOM-BC paper to compare ANCOM-BC, CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, and TSS on the global gut microbiota data.

## When to use

Decision support for choosing a normalization method when you have a preprocessed feature table with two known sample groups and need an objective, visualization-linked ranking of competing normalizations via NMDS separation.

## When NOT to use

- When the question is differential abundance testing with p-values rather than ranking normalization methods for between-group separation
- When samples cannot be assigned to discrete groups — ANCOM-BC v1.0 only supports discrete group.var and BSS here is computed between group levels
- When the goal is multi-group analysis rather than a two-group (Malawi/Venezuela-style) comparison; perNum permutations only activate for more than 2 groups and the reported ranking is for two groups

## Inputs

- Preprocessed OTU/feature table (taxa in rows, samples in columns) from feature_table_pre_process with group labels (e.g., nationality: Malawi vs Venezuela)
- Eight normalized versions of the same table: ANCOM-BC bias-corrected absolute abundances, CSS (metagenomeSeq), MED (DESeq2), UQ (edgeR), TMM (edgeR), ELib-UQ, ELib-TMM, TSS

## Outputs

- NMDS first-two-coordinate ordination per normalization method
- Ranked BSS table written to bss_ranking.csv with columns (method, BSS, rank)

## How to apply

First obtain each candidate normalized table (e.g., ANCOM-BC bias-corrected absolute abundances from ANCOM_BC with adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05; CSS via metagenomeSeq; MED via DESeq2; UQ and TMM via edgeR; plus ELib-UQ, ELib-TMM, and TSS) applied to the same feature_table_pre_process output (sample.var='sample', zero.cut=0.90, lib.cut=1000, neg.lb=TRUE). For each of the eight normalized tables, compute the first two NMDS ordination coordinates. On these coordinates, compute the Between-Group Sum of Squares (BSS) quantifying group separation (Malawi vs Venezuela), then rank methods from largest to smallest BSS. The rationale is that a normalization that better removes sampling-fraction bias yields samples whose group centroids are more separated in ordination space, which is measurable as larger BSS. Write the ranked table (method, BSS, rank) to bss_ranking.csv and check the ordering against the reported result: ANCOM-BC largest, followed by ELib-TMM and MED.

## Related tools

- **ANCOM-BC** (Provides the bias-corrected absolute abundance estimates (via E-M estimation of sampling fractions) that are ranked against competing normalizations) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **metagenomeSeq** (Supplies the CSS normalization whose NMDS/BSS performance is compared)
- **DESeq2** (Supplies the MED (median) normalization whose NMDS/BSS performance is compared)
- **edgeR** (Supplies the UQ and TMM normalizations whose NMDS/BSS performance is compared)
- **dplyr** (R data-manipulation dependency for the ANCOM-BC scripts used in the workflow) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (R optimization dependency required by the ANCOM-BC code) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **R** (Implementation environment for preprocessing, normalization, NMDS, and BSS computation)

## Examples

```
source('ANCOM_BC.R'); pre <- feature_table_pre_process(feature.table, meta.data, sample.var='sample', group.var='nationality', zero.cut=0.90, lib.cut=1000, neg.lb=TRUE); res <- ANCOM_BC(pre$feature.table, pre$grp.name, pre$grp.ind, pre$structure.zeros, adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05)
```

## Evaluation signals

- The BSS ranking reproduces the reported ordering: ANCOM-BC has the largest BSS, followed by ELib-TMM and MED, with the remaining methods performing poorly
- bss_ranking.csv exists with exactly the columns (method, BSS, rank) and eight rows, one per normalization method
- All eight normalizations were computed on the identical feature_table_pre_process output (zero.cut=0.90, lib.cut=1000, neg.lb=TRUE), so rankings are comparable
- Each method's NMDS ordination yields exactly two coordinates per sample with no missing samples relative to the preprocessed table

## Limitations

- BSS on NMDS coordinates is a visualization-linked heuristic for group separation, not a statistical test with p-values or FDR control
- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group; FDR is not controlled in such cases
- Slightly elevated FDR may occur when the proportion of differentially abundant taxa is too large

## Evidence

- [results] ANCOM-BC achieves the largest between-group sum of squares (BSS), followed by ELib-TMM and MED, with the remaining methods performing poorly.: [evidence span withheld — anchor status: not_found]
- [methods] feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb): "feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb)"
- [methods] ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha): "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [abstract] ANCOM-BC estimates the unknown sampling fractions and corrects the bias induced by their differences among samples.: "ANCOM-BC), which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples."
- [readme] Samples with library size less than lib.cut are not included in the analysis.: "Samples with library size less than lib.cut are not included in the analysis."
