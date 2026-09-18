---
name: phyloseq-data-handling
description: Use when the input is a phyloseq object or microbiome count data requiring
  extraction of the absolute abundance OTU table, metadata, and taxonomy before compositional
  differential-abundance testing with ANCOM-BC.
when_to_use_negative:
- Data are already relative abundances or otherwise pre-processed — ANCOM-BC requires
  the observed absolute abundance OTU table as input
- The main variable of interest is continuous — ANCOM-BC v1.0 only supports discrete
  group.var
- Sample sizes are very small (e.g., n = 5 per group) — ANCOM-BC may not perform well
  and FDR is not controlled in that regime
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Environment in which the phyloseq object is loaded and feature_table_pre_process/ANCOM_BC
    are executed
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: dplyr
  role: R-package dependency required by the ANCOM-BC code archive for data manipulation
    during preprocessing
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: nloptr
  role: R-package dependency required by the ANCOM-BC code archive for the optimization
    underlying bias estimation
  repo: github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- name: ANCOM-BC
  role: Downstream differential-abundance method consuming the pre-processed absolute
    abundance table, group.name, group.ind, and structure.zeros
  repo: FrederickHuangLin/ANCOMBC
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17041-7
    title: Analysis of compositions of microbiomes with bias correction
  source_content_hash: sha256:d39f9ae08594cccb425fbd0e37ecf828b6ec7030a9ee1925916724b27f83274b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
  tools:
  - ANCOM-BC
  - R
  - dplyr
  - nloptr
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# phyloseq-data-handling

## Summary

Extract and prepare absolute-abundance OTU tables, sample metadata, and taxonomy from a phyloseq object (e.g., the global gut microbiota dataset) so they can feed into the ANCOM-BC feature_table_pre_process and ANCOM_BC pipeline.

## When to use

Use when the input is a phyloseq object or microbiome count data requiring extraction of the absolute abundance OTU table, metadata, and taxonomy before compositional differential-abundance testing with ANCOM-BC.

## When NOT to use

- Data are already relative abundances or otherwise pre-processed — ANCOM-BC requires the observed absolute abundance OTU table as input
- The main variable of interest is continuous — ANCOM-BC v1.0 only supports discrete group.var
- Sample sizes are very small (e.g., n = 5 per group) — ANCOM-BC may not perform well and FDR is not controlled in that regime

## Inputs

- phyloseq object (otu_table, sample_data, tax_table) or OTU count table with taxa in rows and samples in columns
- Sample metadata data frame with sample ID column (sample.var) and a discrete group variable (group.var, e.g., nationality or age group)
- Thresholds: zero.cut (e.g., 0.90), lib.cut (e.g., 1000), neg.lb = TRUE

## Outputs

- Pre-processed feature.table (data frame of absolute abundances)
- library.size numeric vector
- group.name character vector of group levels
- group.ind numeric group assignment per sample
- structure.zeros matrix of 0/1 flagging structural zeros per taxon per group

## How to apply

Load the dataset as a phyloseq object in R (the ANCOM-BC example code loads data via data(dietswap), and the global gut microbiota dataset is the illustrative case), then extract otu_table, sample_data, and tax_table. Keep OTUs/taxa in rows and samples in columns, as required by feature_table_pre_process, and ensure the metadata contains a sample ID column and a discrete group variable such as 'nationality' or 'bmi_group'. Preprocess with feature_table_pre_process using thresholds like zero.cut = 0.90, lib.cut = 1000, and neg.lb = TRUE so taxa with >90% zeroes are dropped, samples with library size < 1000 are excluded, and structural zeros are flagged via the asymptotic lower bound. The absolute abundance table (not relative abundance) is essential because ANCOM-BC models absolute abundances in a linear regression framework with a sampling-fraction offset; the function returns the pre-processed feature.table, library.size, group.name, group.ind, and structure.zeros needed by ANCOM_BC.

## Related tools

- **R** (Environment in which the phyloseq object is loaded and feature_table_pre_process/ANCOM_BC are executed) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **dplyr** (R-package dependency required by the ANCOM-BC code archive for data manipulation during preprocessing) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **nloptr** (R-package dependency required by the ANCOM-BC code archive for the optimization underlying bias estimation) — github.com/FrederickHuangLin/ANCOM-BC-Code-Archive
- **ANCOM-BC** (Downstream differential-abundance method consuming the pre-processed absolute abundance table, group.name, group.ind, and structure.zeros) — FrederickHuangLin/ANCOMBC

## Examples

```
library(dplyr); library(nloptr); otu_absolute = as.data.frame(otu_table(ps)); pre = feature_table_pre_process(otu_absolute, sample_data(ps), 'sample', 'nationality', zero.cut=0.90, lib.cut=1000, neg.lb=TRUE)
```

## Evaluation signals

- No taxon in the pre-processed feature.table has a proportion of zeroes greater than zero.cut (e.g., 0.90)
- All retained samples have library size >= lib.cut (e.g., 1000); excluded samples are absent from both feature.table and metadata
- structure.zeros is a binary (0/1) matrix with dimensions matching taxa × groups, consistent with neg.lb classification
- group.ind values map 1:1 onto the levels of group.name and each sample's metadata group label
- Taxa remain in rows and samples in columns, and values are absolute counts as expected by ANCOM_BC

## Limitations

- ANCOM-BC v1.0 only supports discrete group.var, so the group variable must be categorical
- ANCOM-BC may not perform well with very small sample sizes, such as n = 5 per group, where FDR is not controlled
- Slightly elevated FDR can occur when the proportion of differentially abundant taxa is too large
- Taxa identified as structural zeros are not suitable for the standard DA test and are handled separately

## Evidence

- [abstract] Absolute abundance modeling rationale: "The absolute abundance data are modeled using a linear regression framework."
- [intro] Why absolute (not relative) abundances must be extracted: "Testing hypotheses regarding mean relative abundance is not equivalent to testing hypotheses regarding mean absolute abundance"
- [methods] Preprocessing function signature used on the phyloseq-extracted table: "feature_table_pre_process(feature.table, meta.data, sample.var, group.var, zero.cut, lib.cut, neg.lb)"
- [readme] Zero filter applied during preprocessing: "Taxa with proportion of zeroes greater than zero.cut are not included in the analysis."
- [readme] Library-size filter applied during preprocessing: "Samples with library size less than lib.cut are not included in the analysis."
