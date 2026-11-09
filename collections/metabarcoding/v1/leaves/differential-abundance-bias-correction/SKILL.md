---
name: differential-abundance-bias-correction
description: Use when comparing absolute taxon abundances across experimental groups
  from compositional OTU/feature tables (samples × taxa counts) where unequal sampling
  fractions may bias relative-abundance-based tests.
when_to_use_negative:
- Sample sizes are very small (e.g. n = 5 per group), where ANCOM-BC may not perform
  well and FDR is not controlled
- The taxon of interest is a structural zero in a group (presence/absence test), since
  ANCOM-BC/BC2 are not designed to detect significant differences in taxa with structural
  zeros
- The goal is a multi-group or covariate-adjusted model beyond ANCOM-BC v1.0's discrete
  two-group/multi-group group.var support — use the newer ANCOMBC package functions
  instead
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_2269
tools:
- name: ANCOM-BC
  role: 'Bias-corrected differential abundance method: estimates sampling fractions
    and tests mean differences of absolute abundance via E-M and a two-sided Z-test'
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOMBC
  role: R/Bioconductor package implementing ancombc and related DA/correlation analyses
    (ancom, ancombc2, secom)
  repo: https://github.com/FrederickHuangLin/ANCOMBC
- name: dplyr
  role: Required R dependency for the ANCOM-BC scripts
- name: nloptr
  role: Required R dependency for the ANCOM-BC scripts
- name: metagenomeSeq
  role: Provides CSS normalization used as a competing normalization in the bias-correction
    comparison
- name: DESeq2
  role: Provides the Median (MED) normalization used as a competing normalization
    in the comparison
- name: edgeR
  role: Provides UQ and TMM normalizations used as competing normalizations in the
    comparison
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:3d1bf74f0e20063230cbaee538513869316253e3b4d6f089bac1a75af64b0e51
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - ANCOMBC
  - DESeq2
  - R
  - dplyr
  - edgeR
  - metagenomeSeq
  - nloptr
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.47058823529411764
n_claims: 17
---

# differential-abundance-bias-correction

## Summary

Applies ANCOM-BC to estimate sample-specific sampling fractions via an E-M algorithm and correct the bias they induce in differential abundance analysis of microbiome count data, within a linear regression framework on the log scale.

## When to use

Use when comparing absolute taxon abundances across experimental groups from compositional OTU/feature tables (samples × taxa counts) where unequal sampling fractions may bias relative-abundance-based tests.

## When NOT to use

- Sample sizes are very small (e.g. n = 5 per group), where ANCOM-BC may not perform well and FDR is not controlled
- The taxon of interest is a structural zero in a group (presence/absence test), since ANCOM-BC/BC2 are not designed to detect significant differences in taxa with structural zeros
- The goal is a multi-group or covariate-adjusted model beyond ANCOM-BC v1.0's discrete two-group/multi-group group.var support — use the newer ANCOMBC package functions instead

## Inputs

- Feature table (data frame or matrix) of observed OTU counts with taxa in rows and samples in columns
- Metadata data frame containing sample IDs and a discrete group variable (group.var) such as 'nationality'

## Outputs

- Pre-processed feature table, library sizes, group names/indices, and structural-zero matrix from feature_table_pre_process
- ANCOM_BC result data frame with mean.difference, se, W, p.val, q.val and diff.abn per taxon, plus estimated log sampling fractions (d), group means (mu), bias.em and bias.wls

## How to apply

First run feature_table_pre_process on the feature table and metadata with sample.var, group.var, zero.cut, lib.cut and neg.lb (e.g. zero.cut=0.90, lib.cut=1000, neg.lb=TRUE) to drop taxa with excessive zeroes and low-library-size samples and flag structural zeros. Then call ANCOM_BC with grp.name and grp.ind from preprocessing, passing struc.zero, adj.method (e.g. 'bonferroni'), tol.EM=1e-5, max.iterNum=100, perNum=1000 and alpha=0.05; the E-M algorithm estimates the log-scale sampling fraction offset d per sample, and a two-sided Z-test on W = mean.difference/se yields p.val and adjusted q.val. The same preprocessed table can be re-normalized with competing methods (CSS via metagenomeSeq, MED via DESeq2, UQ/TMM via edgeR, plus ELib-UQ, ELib-TMM, TSS) for benchmarking, e.g. ranking methods by Between-Group Sum of Squares (BSS) on NMDS coordinates. The R implementation depends on dplyr and nloptr, as stated in the code archive.

## Related tools

- **ANCOM-BC** (Bias-corrected differential abundance method: estimates sampling fractions and tests mean differences of absolute abundance via E-M and a two-sided Z-test) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOMBC** (R/Bioconductor package implementing ancombc and related DA/correlation analyses (ancom, ancombc2, secom)) — https://github.com/FrederickHuangLin/ANCOMBC
- **dplyr** (Required R dependency for the ANCOM-BC scripts)
- **nloptr** (Required R dependency for the ANCOM-BC scripts)
- **metagenomeSeq** (Provides CSS normalization used as a competing normalization in the bias-correction comparison)
- **DESeq2** (Provides the Median (MED) normalization used as a competing normalization in the comparison)
- **edgeR** (Provides UQ and TMM normalizations used as competing normalizations in the comparison)

## Examples

```
feature_table_pre_process(feature.table, meta.data, sample.var="sample", group.var="nationality", zero.cut=0.90, lib.cut=1000, neg.lb=TRUE)
```

## Evaluation signals

- q.val for each taxon equals adj.method (e.g. 'bonferroni') applied to the two-sided Z-test p.val, and diff.abn is TRUE where q.val < alpha (0.05)
- The E-M loop converged: tol.EM=1e-5 reached within max.iterNum=100 iterations, and bias.em (sampling-fraction mean difference) is finite and consistent with bias.wls
- Preprocessing invariants: taxa with proportion of zeroes > zero.cut and samples with library size < lib.cut are absent from the returned feature table
- In the NMDS/BSS benchmark on Malawi vs Venezuela global gut samples, ANCOM-BC achieves the largest BSS, followed by ELib-TMM and MED, reproducing the reported ordering

## Limitations

- ANCOM-BC may not perform well when sample sizes are very small, such as n = 5 per group; FDR is not controlled in such cases
- FDR may be slightly elevated when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports a discrete group.var; the E-M permutation argument perNum is active only when there exist more than 2 groups

## Evidence

- [abstract] which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples.: "which estimates the unknown sampling fractions and corrects the bias induced by their differences among samples."
- [intro] Testing mean relative vs absolute abundance: "Testing hypotheses regarding mean relative abundance is not equivalent to testing hypotheses regarding mean absolute abundance"
- [methods] feature_table_pre_process signature: "feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb)"
- [methods] ANCOM_BC signature: "ANCOM_BC(feature.table, grp.name, grp.ind, struc.zero, adj.method, tol.EM, max.iterNum, perNum, alpha)"
- [readme] Code archive scope statement: "The current code implements ANCOM-BC in cross-sectional datasets for comparing the change of absolute abundance for each taxon among different experimental groups."
