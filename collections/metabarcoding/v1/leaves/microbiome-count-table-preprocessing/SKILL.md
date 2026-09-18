---
name: microbiome-count-table-preprocessing
description: Use when raw 16S rRNA ASV count tables must be filtered of rare taxa
  prior to differential abundance analysis, and the filter must be independent of
  any test statistic (Independent Filtering).
when_to_use_negative:
- The feature table is already prevalence-filtered or the downstream tool performs
  its own zero-handling pre-processing (e.g. ANCOM's feature_table_pre_process with
  zero_cut = 0.90), risking double filtering
- The analysis goal is rarefaction or library-size correction rather than rare-taxon
  removal — that is handled separately via the -D rarefaction depth parameter, not
  the -F prevalence filter
- Datasets contain more than 2 sample groupings, since the metadata file for run_all_tools.sh
  supports a maximum of 2 groupings
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: phyloseq
  role: R package used to load the ASV count table with paired sample metadata and
    apply filter_taxa at the 0.1 prevalence threshold (version 1.29.0)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_all_tools.sh
  role: Pipeline script whose -F 0.1 flag triggers the 10% prevalence filtering of
    the ASV table before running all DA tools on a dataset
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_0.1_filt_datasets.sh
  role: Wrapper script that runs all DA tools on the 38 prevalence-filtered datasets
    produced by this skill
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:fff58a4da72baf866177e906c17eed5d82bea1b0b9f7f6c3f4109a1aa9dd71f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - phyloseq
  - run_0.1_filt_datasets.sh
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# microbiome-count-table-preprocessing

## Summary

Pre-processes 16S rRNA ASV abundance tables by applying an independent 10% prevalence filter (removing ASVs present in fewer than 10% of samples per dataset) before differential abundance testing, producing the filtered abundance tables consumed by downstream DA tools.

## When to use

Use when raw 16S rRNA ASV count tables must be filtered of rare taxa prior to differential abundance analysis, and the filter must be independent of any test statistic (Independent Filtering).

## When NOT to use

- The feature table is already prevalence-filtered or the downstream tool performs its own zero-handling pre-processing (e.g. ANCOM's feature_table_pre_process with zero_cut = 0.90), risking double filtering
- The analysis goal is rarefaction or library-size correction rather than rare-taxon removal — that is handled separately via the -D rarefaction depth parameter, not the -F prevalence filter
- Datasets contain more than 2 sample groupings, since the metadata file for run_all_tools.sh supports a maximum of 2 groupings

## Inputs

- Raw 16S rRNA ASV abundance table (tsv, taxa x samples) from the figshare 16S rRNA Microbiome Datasets deposit (https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724)
- Two-column metadata tsv (sample names in column 1, sample groupings in column 2, maximum 2 groupings)
- Prevalence threshold (0.1, passed as -F 0.1 to run_all_tools.sh)

## Outputs

- Per-dataset prevalence-filtered ASV abundance table in the same tsv structure as the input (38 filtered datasets total)
- Filtered tables retaining only ASVs present in at least 10% of samples, with sample order matching the metadata grouping column

## How to apply

Load each raw ASV abundance table into R with phyloseq (v1.29.0), pairing sample_data with a two-column metadata tsv whose first column holds sample names and second column holds the groupings (maximum 2 groupings). Compute each ASV's prevalence as the fraction of samples with a non-zero count, then use phyloseq filter_taxa to remove any ASV with prevalence below the 0.1 threshold (the -F 0.1 setting of run_all_tools.sh), keeping sample order identical to the metadata file. Apply the filter independently of any test statistic so that DA testing is not biased by data-dependent feature selection. Write the filtered abundance table per dataset in the same tsv structure as the input, yielding the 38 filtered datasets consumed by run_0.1_filt_datasets.sh. The 10% cut-off was arbitrarily selected in the manuscript; the authors note more work is needed to estimate an optimal prevalence filter cut-off.

## Related tools

- **phyloseq** (R package used to load the ASV count table with paired sample metadata and apply filter_taxa at the 0.1 prevalence threshold (version 1.29.0)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_all_tools.sh** (Pipeline script whose -F 0.1 flag triggers the 10% prevalence filtering of the ASV table before running all DA tools on a dataset) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_0.1_filt_datasets.sh** (Wrapper script that runs all DA tools on the 38 prevalence-filtered datasets produced by this skill) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Every output file exists for all 38 datasets
- Each filtered table retains every sample present in the metadata grouping column
- Every retained ASV appears in at least 10% of samples within its dataset
- The filtered table preserves the input tsv structure and the sample order matches the two-column metadata file
- Filtering decisions are independent of any DA test statistic (no feature was removed based on its differential abundance result)

## Limitations

- The 10% prevalence cut-off was arbitrarily selected; the authors acknowledge that more work is needed to estimate an optimal cut-off
- It remains unclear whether filtering rare taxa has much effect on DA results in practice
- Although statistical arguments regarding the validity of independent filtering are beyond the scope of this work
- Whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work

## Evidence

- [readme] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript): "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [methods] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [discussion] we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off rather than just"
- [readme] Note that the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings): "Note that the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings)"
- [readme] All the raw tables along with the correct folder structuring is available in figshare at the following link: https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724: "All the raw tables along with the correct folder structuring is available in figshare at the following link: https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724"
