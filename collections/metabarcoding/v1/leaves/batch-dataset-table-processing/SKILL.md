---
name: batch-dataset-table-processing
description: Use when a repository or pipeline contains many raw ASV/OTU count tables
  plus two-column sample-metadata TSVs and DA tools must be run on identically pre-processed,
  prevalence-filtered tables (e.g. 10% prevalence cut-off) across the full batch.
when_to_use_negative:
- The analysis requires unfiltered tables (e.g. the false-positive or no-filter comparisons
  run via run_no_filt_datasets.sh / run_all_FP_datasets_unfilt.sh)
- Only rarefaction to a fixed depth is needed and no prevalence-based taxon removal
  is wanted (use -D without -F)
- The input is already a filtered/relative-abundance feature table rather than an
  absolute raw ASV count table with paired metadata
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: run_all_tools.sh
  role: Key pipeline script; invoked per dataset with -F 0.1 to apply the 10% prevalence
    filter before running DA tools
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_0.1_filt_datasets.sh
  role: Batch driver that runs all tools on the 38 prevalence-filtered datasets
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: phyloseq
  role: R library (v1.29.0) used to load count tables with metadata and apply filter_taxa
    at the 0.1 prevalence threshold
- name: Docker image dockerdkd/hackathon2021
  role: Prebuilt image providing correct versions of all tools so the batch scripts
    run reproducibly
  repo: https://hub.docker.com/r/dockerdkd/hackathon2021
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:6ad36e9f040410e90927d34417dbe355fa9023df98e6525a5e6aabff6be99ea3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - Docker image dockerdkd/hackathon2021
  - phyloseq
  - run_0.1_filt_datasets.sh
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 6
---

# batch-dataset-table-processing

## Summary

Batch-process multiple 16S rRNA ASV abundance tables (38 datasets) into prevalence-filtered versions by removing ASVs present in fewer than 10% of samples within each dataset, using run_all_tools.sh with the -F 0.1 flag and phyloseq-based filtering so all downstream differential abundance tools consume identically pre-processed tables.

## When to use

Use when a repository or pipeline contains many raw ASV/OTU count tables plus two-column sample-metadata TSVs and DA tools must be run on identically pre-processed, prevalence-filtered tables (e.g. 10% prevalence cut-off) across the full batch.

## When NOT to use

- The analysis requires unfiltered tables (e.g. the false-positive or no-filter comparisons run via run_no_filt_datasets.sh / run_all_FP_datasets_unfilt.sh)
- Only rarefaction to a fixed depth is needed and no prevalence-based taxon removal is wanted (use -D without -F)
- The input is already a filtered/relative-abundance feature table rather than an absolute raw ASV count table with paired metadata

## Inputs

- Raw 16S rRNA ASV abundance tables (TSV) from the figshare deposit https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724
- Two-column metadata TSV files (column 1: sample names; column 2: sample groupings, max 2 groups)
- config.sh with DATA_DIR, ANCOM_DIR, and TOOL_DIR parameters set to the unzipped Hackathon data and script folders

## Outputs

- Prevalence-filtered abundance tables (one per dataset, 38 total) in the same TSV structure as the inputs, with each retained ASV present in at least 10% of samples
- Batch outputs consumed by run_0.1_filt_datasets.sh for downstream differential abundance testing

## How to apply

Download the raw 16S rRNA ASV tables and two-column metadata files from the figshare deposit (16S rRNA Microbiome Datasets, https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724) and point config.sh DATA_DIR at the unzipped Hackathon folder. For each dataset, load the count table into R with phyloseq (sample_data paired with the metadata TSV mapping sample names to groupings) and compute each ASV's prevalence as the fraction of samples with a non-zero count. Remove any ASV with prevalence below 0.1 via phyloseq filter_taxa, keeping sample order identical to the metadata file and applying the filter independently of any test statistic (Independent Filtering). In practice this is the -F 0.1 setting of run_all_tools.sh, optionally with -D 2000 for rarefaction depth; run_0.1_filt_datasets.sh applies it across all 38 datasets to feed the filtered DA analyses. Write each filtered table out in the same TSV structure as its input.

## Related tools

- **run_all_tools.sh** (Key pipeline script; invoked per dataset with -F 0.1 to apply the 10% prevalence filter before running DA tools) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_0.1_filt_datasets.sh** (Batch driver that runs all tools on the 38 prevalence-filtered datasets) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **phyloseq** (R library (v1.29.0) used to load count tables with metadata and apply filter_taxa at the 0.1 prevalence threshold)
- **Docker image dockerdkd/hackathon2021** (Prebuilt image providing correct versions of all tools so the batch scripts run reproducibly) — https://hub.docker.com/r/dockerdkd/hackathon2021

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Each of the 38 filtered dataset files exists and retains the same TSV structure as its unfiltered input
- Every sample present in the metadata grouping column is retained in each filtered table (sample order identical to metadata file)
- Every retained ASV appears in at least 10% of samples within its dataset (prevalence >= 0.1)
- The filtering was applied independently of any test statistic (Independent Filtering), so retained taxa do not depend on group-difference statistics
- Expected behavior can be cross-checked against the included Misc_datafiles/Expected_MALL_output for the Human-ALL (MALL) dataset

## Limitations

- The 10% prevalence cut-off was arbitrarily selected; the optimal filter threshold is unknown and the authors note more work is needed to estimate an optimal cut-off
- Whether rarefying (or filtering) is advisable for other analyses, such as computing diversity metrics, is beyond the scope of this work
- Results of downstream DA tools depend on data pre-processing, so filtering choices directly change which ASVs are called significant

## Evidence

- [readme] 10% prevalence filter flag: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [readme] batch script over 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [discussion] recommendation to prevalence filter before DA testing: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off rather than just"
- [readme] raw data deposit location: "All the raw tables along with the correct folder structuring is available in figshare at the following link: https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724"
- [readme] per-dataset command with filtering: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
