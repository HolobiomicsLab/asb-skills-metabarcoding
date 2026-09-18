---
name: asv-prevalence-filtering
description: Use when preparing raw 16S rRNA ASV abundance tables (tsv count tables
  plus two-column sample-group metadata) for multi-tool differential abundance testing,
  so that DA results are not driven by rare taxa.
when_to_use_negative:
- Input is already a prevalence-filtered or relative-abundance table (prevalence computed
  on transformed tables is meaningless; ANCOM pre-processing explicitly expects absolute
  abundance)
- The metadata has more than two sample groupings — the pipeline metadata format specifies
  'maximum 2 groupings'
- The goal is advice on rarefying for other analyses such as diversity metrics, which
  the article explicitly states is beyond the scope of this work
edam_operation: http://edamontology.org/operation_3675
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: phyloseq
  role: R package used to load count tables with sample_data metadata and execute
    the filter_taxa call that removes ASVs below the 0.1 prevalence threshold
- name: run_all_tools.sh
  role: Pipeline driver script whose -F flag sets the prevalence filter (-F 0.1 =
    10% prevalence filter) and whose -D flag sets the rarefaction depth (2000)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_0.1_filt_datasets.sh
  role: Batch script that consumes the 38 prevalence-filtered datasets and runs all
    DA tools on them
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
merged_aliases:
- asv-count-table-prevalence-filtering
merged_alias_records:
- alias: asv-count-table-prevalence-filtering
  slug: asv-count-table-prevalence-filtering
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:9b49181472a937fe587865161a1ab1a7b57f322e31fb9ddd7fe45ab44b630687
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - ALDEx2
  - ANCOM
  - ANCOM-II
  - DESeq2
  - LEfSe
  - corncob
  - edgeR
  - limma voom
  - metagenomeSeq
  - phyloseq
  - run_0.1_filt_datasets.sh
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_3675
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# asv-prevalence-filtering

## Summary

Pre-process raw 16S rRNA ASV count tables by removing ASVs observed in fewer than 10% of samples within each dataset, producing prevalence-filtered abundance tables used as inputs to differential abundance (DA) testing in the 38-dataset hackathon comparison.

## When to use

Use when preparing raw 16S rRNA ASV abundance tables (tsv count tables plus two-column sample-group metadata) for multi-tool differential abundance testing, so that DA results are not driven by rare taxa.

## When NOT to use

- Input is already a prevalence-filtered or relative-abundance table (prevalence computed on transformed tables is meaningless; ANCOM pre-processing explicitly expects absolute abundance)
- The metadata has more than two sample groupings — the pipeline metadata format specifies 'maximum 2 groupings'
- The goal is advice on rarefying for other analyses such as diversity metrics, which the article explicitly states is beyond the scope of this work

## Inputs

- Raw 16S rRNA ASV abundance tables (tsv) from the figshare deposit 16S rRNA Microbiome Datasets (https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724)
- Two-column metadata tsv files (first column sample names, second column sample groupings, maximum 2 groupings)

## Outputs

- Per-dataset prevalence-filtered ASV abundance tables in the same tsv structure as the input (the 38 filtered datasets)
- Filtered datasets consumable by run_0.1_filt_datasets.sh for downstream DA tool runs

## How to apply

For each dataset, load the ASV count table into R with phyloseq (sample_data paired with the two-column tsv metadata that maps sample names to groupings) and compute each ASV's prevalence as the fraction of samples with a non-zero count. Remove every ASV with prevalence below the 0.1 threshold using phyloseq filter_taxa, keeping the sample order identical to the metadata file, and apply the filter independently of any test statistic (Independent Filtering). Write the filtered table in the same tsv structure as the input, producing the 38 filtered datasets consumed by run_0.1_filt_datasets.sh. In the pipeline this filter is invoked via the -F 0.1 setting of run_all_tools.sh, optionally alongside a -D 2000 rarefaction depth; the article recommends prevalence filtering (e.g., at 10%) prior to DA testing while noting the cut-off is arbitrary.

## Related tools

- **phyloseq** (R package used to load count tables with sample_data metadata and execute the filter_taxa call that removes ASVs below the 0.1 prevalence threshold)
- **run_all_tools.sh** (Pipeline driver script whose -F flag sets the prevalence filter (-F 0.1 = 10% prevalence filter) and whose -D flag sets the rarefaction depth (2000)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_0.1_filt_datasets.sh** (Batch script that consumes the 38 prevalence-filtered datasets and runs all DA tools on them) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Every output file exists for all 38 filtered datasets
- Every retained ASV appears in at least 10% of samples within its dataset (prevalence >= 0.1)
- Every output retains every sample present in the metadata grouping column, with sample order identical to the metadata file
- Filtering is applied independently of any test statistic (Independent Filtering), before any DA tool is invoked
- Re-running the 10% filter yields the same tsv structure as the input tables

## Limitations

- The 10% prevalence threshold was arbitrarily selected; the article notes 'more work is needed to estimate an optimal cut-off rather than just arbitrarily selecting one'
- The filtered results depend on data pre-processing, so conclusions differ between the filtered (run_0.1_filt_datasets.sh) and unfiltered (run_no_filt_datasets.sh) pipelines
- The 10% filter was chosen for two-group DA testing on 16S rRNA ASV tables; its adequacy for other designs or data types is not established by this study

## Evidence

- [readme] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript): "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [discussion] we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off rather than just"
- [readme] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [abstract] we compare the performance of 14 differential abundance testing methods on 38 16S rRNA gene datasets with two sample groups: "we compare the performance of 14 differential abundance testing methods on 38 16S rRNA gene datasets with two sample groups"
