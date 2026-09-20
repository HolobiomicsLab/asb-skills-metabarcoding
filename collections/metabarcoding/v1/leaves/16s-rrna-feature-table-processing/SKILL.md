---
name: 16s-rrna-feature-table-processing
description: Use when a raw 16S rRNA ASV/OTU count table (taxa in rows, samples in
  columns) plus sample metadata must be prepared for DA testing — e.g., before applying
  run_all_tools.sh or ANCOM — and decisions on prevalence filtering, rarefaction depth,
  and zero treatment are needed.
when_to_use_negative:
- The input is already a relative abundance table (column totals = 1) — ANCOM's ANCOM()
  and feature_table_pre_process() require the absolute abundance table and will misbehave
  on relative data.
- The question is choosing/running a specific DA method on an already-processed table
  — this skill covers only the preprocessing/filtering stage (e.g., run_0.1_filt_datasets.sh
  vs run_no_filt_datasets.sh comes first).
- The data are not count-based 16S rRNA feature tables with two sample groups — the
  10% prevalence filter and rarefaction settings were tuned for the 38 16S datasets
  in this comparison.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: ANCOM-II (feature_table_pre_process)
  role: Zero-handling and preprocessing of the absolute feature table (out_cut=0.05,
    zero_cut=0.90, lib_cut, neg_lb) before DA testing
  repo: FrederickHuangLin/ANCOM-Code-Archive
- name: run_all_tools.sh pipeline
  role: Applies -D rarefaction depth and -F prevalence filtering flags and runs all
    14 DA tools on the prepared table
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image
- name: ALDEx2
  role: Consumer of the preprocessed tables; operates on relative abundance and CLR-transformed
    inputs (version 1.18.0 in the pipeline)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:81509fa1e093f56f84460239b32c30e3611278ec99c8fc4dd8c6f988924d2f53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
  tools:
  - ALDEx2
  - ANCOM-II (feature_table_pre_process)
  - ROCR
  - dplyr
  - ggplot2
  - run_all_tools.sh pipeline
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# 16s-rrna-feature-table-processing

## Summary

Preprocesses 16S rRNA ASV/OTU read-count tables so they are suitable inputs for the 14 differential abundance tools compared in this study, covering prevalence filtering, rarefaction, zero handling, and absolute vs relative vs CLR abundance representation.

## When to use

Use when a raw 16S rRNA ASV/OTU count table (taxa in rows, samples in columns) plus sample metadata must be prepared for DA testing — e.g., before applying run_all_tools.sh or ANCOM — and decisions on prevalence filtering, rarefaction depth, and zero treatment are needed.

## When NOT to use

- The input is already a relative abundance table (column totals = 1) — ANCOM's ANCOM() and feature_table_pre_process() require the absolute abundance table and will misbehave on relative data.
- The question is choosing/running a specific DA method on an already-processed table — this skill covers only the preprocessing/filtering stage (e.g., run_0.1_filt_datasets.sh vs run_no_filt_datasets.sh comes first).
- The data are not count-based 16S rRNA feature tables with two sample groups — the 10% prevalence filter and rarefaction settings were tuned for the 38 16S datasets in this comparison.

## Inputs

- 16S rRNA ASV/OTU absolute read-count table (taxa in rows, samples in columns; TSV)
- Sample metadata TSV mapping sample IDs to group variables
- Raw tables and folder structure from the figshare 16S rRNA Microbiome Datasets archive (14531724)
- config.sh with DATA_DIR, ANCOM_DIR, and TOOL_DIR PATHs

## Outputs

- Filtered (e.g., 10% prevalence) and optionally rarefied ASV tables fed to run_all_tools.sh
- Preprocessed feature table, metadata, and structural-zero matrix from feature_table_pre_process
- Per-dataset tool-ready abundance inputs in both relative-abundance and CLR representations

## How to apply

Start from an absolute (not relative) abundance table; the ANCOM archive explicitly warns not to supply a relative abundance table whose column totals equal 1. Apply prevalence filtering with the paper's setting -F 0.1 (a 10% prevalence filter, as used in the manuscript's analysis of the 38 filtered datasets) and, if subsampling, set the rarefaction depth via -D (e.g., -D 2000), then launch all tools with run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER. For ANCOM specifically, pass the table through feature_table_pre_process(feature_table, meta_data, sample_var, group_var, out_cut=0.05, zero_cut=0.90, lib_cut, neg_lb), where taxa with a proportion of zeroes above zero_cut are dropped and structural zeros are detected per ANCOM-II section 3.2. Supply relative-abundance and CLR-transformed versions of the filtered tables only when downstream analyses (e.g., AUROC discrimination of significant ASVs) require both input types. Filter independently of the test statistic and before DA testing, since the authors recommend prevalence filtering (e.g., at 10%) of features prior to DA testing.

## Related tools

- **ANCOM-II (feature_table_pre_process)** (Zero-handling and preprocessing of the absolute feature table (out_cut=0.05, zero_cut=0.90, lib_cut, neg_lb) before DA testing) — FrederickHuangLin/ANCOM-Code-Archive
- **run_all_tools.sh pipeline** (Applies -D rarefaction depth and -F prevalence filtering flags and runs all 14 DA tools on the prepared table) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image
- **ALDEx2** (Consumer of the preprocessed tables; operates on relative abundance and CLR-transformed inputs (version 1.18.0 in the pipeline)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image

## Examples

```
bash run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- The prevalence filter flag is set to the manuscript value (-F 0.1, i.e., a 10% prevalence filter) and rarefaction depth (-D) is explicitly chosen when subsampling.
- The table passed to ANCOM() / feature_table_pre_process() remains an absolute abundance table; no relative transformation was applied beforehand.
- Taxa with a proportion of zeroes greater than zero_cut (default 0.90) are excluded from the preprocessed output, and the structural-zero matrix is returned when group_var is provided.
- run_all_tools.sh completes on all 38 datasets, matching the outputs that run_0.1_filt_datasets.sh and run_no_filt_datasets.sh are designed to generate.
- Downstream DA results depend on the preprocessing choices (filtered vs unfiltered), so the significant-ASV tables trace back to the documented filter settings.

## Limitations

- The optimal prevalence filter cut-off is unknown; the 10% threshold was arbitrarily selected and more work is needed to estimate an optimal cut-off.
- Whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work, and rarefaction has been criticized for reducing statistical power and yielding high false positive rates.
- The number of significant features identified by downstream tools correlates with sample size, sequencing depth, and effect size of community differences, so preprocessing choices propagate into tool performance.

## Evidence

- [methods] run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] -F represents the prevalence filtering: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [readme] ANCOM requires absolute abundance tables: [evidence span withheld — anchor status: not_found]
- [readme] zero_cut drops taxa with too many zeroes: [evidence span withheld — anchor status: not_found]
- [discussion] 10% prevalence filtering recommendation: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing"
- [abstract] results depend on data pre-processing: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
