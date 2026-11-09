---
name: nmds-ordination
description: Use when a user needs to compare competing normalization methods (ANCOM-BC,
  CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, TSS) on compositional count data by visualizing
  group separation in NMDS space and quantifying it with BSS.
when_to_use_negative:
- The input is differential abundance p-values or effect sizes rather than a raw feature
  count table — this skill compares normalizations, not DA results
- The goal is statistical DA testing with p-values and confidence intervals; use the
  ANCOM-BC two-sided Z-test workflow directly instead of NMDS/BSS ranking
- Sample sizes are very small (e.g. n = 5 per group), where ANCOM-BC and the downstream
  ordination comparison are not expected to perform reliably
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: ANCOM-BC
  role: Provides the bias-corrected absolute abundance estimates via E-M estimation
    of sampling fractions; one of the eight normalizations ranked by BSS
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: metagenomeSeq
  role: Implements the CSS normalization of the preprocessed feature table
- name: DESeq2
  role: Implements the Median (MED) normalization of the preprocessed feature table
- name: edgeR
  role: Implements modified UQ and TMM normalizations used in the comparison
- name: R
  role: Environment for the pipeline; requires dplyr and nloptr libraries per the
    code archive
  repo: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: Data manipulation of the preprocessed feature table and metadata within the
    R workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:a1d5415b021ef7cb2590fa57a8685512f340580cf35e468efd79944f949afb73
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# nmds-ordination

## Summary

Method-rich skill for benchmarking normalization pipelines on microbiome data by ordinating each normalized feature table with NMDS and ranking methods by Between-Group Sum of Squares (BSS), as done in the ANCOM-BC paper's global gut microbiota (Malawi vs Venezuela) comparison.

## When to use

Use when a user needs to compare competing normalization methods (ANCOM-BC, CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, TSS) on compositional count data by visualizing group separation in NMDS space and quantifying it with BSS.

## When NOT to use

- The input is differential abundance p-values or effect sizes rather than a raw feature count table — this skill compares normalizations, not DA results
- The goal is statistical DA testing with p-values and confidence intervals; use the ANCOM-BC two-sided Z-test workflow directly instead of NMDS/BSS ranking
- Sample sizes are very small (e.g. n = 5 per group), where ANCOM-BC and the downstream ordination comparison are not expected to perform reliably

## Inputs

- Raw OTU/feature count table (taxa in rows, samples in columns) from the global gut microbiota dataset
- Metadata data frame with sample IDs and group labels (e.g. nationality: Malawi vs Venezuela)
- Eight normalized feature tables: ANCOM-BC (bias-corrected absolute abundance), CSS, MED, UQ, TMM, ELib-UQ, ELib-TMM, TSS

## Outputs

- NMDS first two ordination coordinates for each normalized table
- Ranked BSS table with columns (method, BSS, rank) written to bss_ranking.csv
- NMDS ordination plots comparing group separation per normalization method

## How to apply

Subset the feature table to the groups of interest (e.g. Malawi/Venezuela samples of the global gut microbiota dataset) and preprocess it with feature_table_pre_process using sample.var='sample', zero.cut=0.90, lib.cut=1000, neg.lb=TRUE, so taxa with >90% zeroes and samples with library size below 1000 are removed. Run ANCOM_BC with adj.method='bonferroni', tol.EM=1e-5, max.iterNum=100, perNum=1000, alpha=0.05 to get bias-corrected absolute abundances via the E-M algorithm, which estimates the sampling fractions that induce the bias. Apply the competing normalizations to the same preprocessed table: CSS via metagenomeSeq, MED via DESeq2, UQ and TMM via edgeR (the paper notes edgeR implements modified versions of UQ and TMM), plus ELib-UQ, ELib-TMM, and TSS. For each of the eight normalized tables compute the first two NMDS ordination coordinates, then compute the Between-Group Sum of Squares (BSS) on those coordinates and rank methods from largest to smallest BSS. The rationale is that a normalization that better removes sampling-fraction bias should yield larger between-group separation relative to within-group scatter on the NMDS axes.

## Related tools

- **ANCOM-BC** (Provides the bias-corrected absolute abundance estimates via E-M estimation of sampling fractions; one of the eight normalizations ranked by BSS) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **metagenomeSeq** (Implements the CSS normalization of the preprocessed feature table)
- **DESeq2** (Implements the Median (MED) normalization of the preprocessed feature table)
- **edgeR** (Implements modified UQ and TMM normalizations used in the comparison)
- **R** (Environment for the pipeline; requires dplyr and nloptr libraries per the code archive) — https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (Data manipulation of the preprocessed feature table and metadata within the R workflow)

## Examples

```
otu_abs <- feature_table_pre_process(feature.table, meta.data, sample.var='sample', group.var='nationality', zero.cut=0.90, lib.cut=1000, neg.lb=TRUE)
```

## Evaluation signals

- The ranked BSS table reproduces the reported ordering: ANCOM-BC has the largest BSS, followed by ELib-TMM and MED, with remaining methods performing poorly
- bss_ranking.csv contains exactly the columns (method, BSS, rank) and one row for each of the eight normalization methods
- The preprocessed table excludes taxa with proportion of zeroes greater than zero.cut (0.90) and samples with library size less than lib.cut (1000)
- Each NMDS solution yields exactly two ordination coordinates per sample, computed identically across the eight normalized tables
- The ANCOM_BC E-M run converges within max.iterNum=100 at tol.EM=1e-5 and returns estimated sampling fractions d

## Limitations

- ANCOM-BC may not perform well with very small sample sizes (e.g. n = 5 per group), so the BSS ranking based on its output is unreliable in such settings
- Slightly elevated FDR can occur when the proportion of differentially abundant taxa is too large
- ANCOM-BC v1.0 only supports discrete group.var, so the NMDS/BSS comparison is limited to categorical grouping of samples

## Evidence

- [intro] we also considered modified versions of UQ and TMM implemented in edgeR11: "we also considered modified versions of UQ and TMM implemented in edgeR11"
- [methods] feature_table_pre_process signature: "feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb)"
- [methods] zero.cut filter definition: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis"
- [discussion] sampling fraction estimation rationale: "it is the only method available in the literature that estimates the sampling fraction and performs DA analysis by correcting bias due to differential sampling fractions across samples"
- [readme] code archive scope statement: "The current code implements ANCOM-BC in cross-sectional datasets for comparing the change of absolute abundance for each taxon among different experimental groups."
