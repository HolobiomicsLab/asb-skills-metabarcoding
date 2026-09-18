---
name: sample-metadata-matching
description: Use when preparing a raw ASV/OTU count table and its metadata file for
  differential abundance analysis and you must verify that every sample name in the
  count table maps to exactly one grouping in the metadata, preserving sample order.
when_to_use_negative:
- The metadata file has more than two groupings — run_all_tools.sh only supports two-column
  metadata with maximum 2 groupings
- Filtering would depend on the test statistic (e.g. filtering significant taxa post-hoc)
  — the prevalence filter must be applied independently of any test statistic
- You need multi-group or covariate-rich metadata handling — this skill only covers
  two-column two-group design tables
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: phyloseq
  role: Loads the ASV count table and pairs it with the two-column metadata tsv via
    sample_data; filter_taxa applies the prevalence filter after sample alignment
- name: run_all_tools.sh
  role: Consumes the metadata file via -G and the ASV table via -A, enforcing the
    two-column tsv metadata format and applying the -F 0.1 prevalence filter on matched
    samples
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_0.1_filt_datasets.sh
  role: Runs all DA tools on the 38 prevalence-filtered datasets produced after metadata-matched
    filtering
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:9df88403ecfdbca1c8447c44cfc21bb7591892de8952bb92649b046461b6df74
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 7
---

# sample-metadata-matching

## Summary

Aligns each 16S rRNA ASV abundance table with its two-column sample metadata tsv (sample names in column 1, sample groupings in column 2, maximum 2 groupings) so downstream prevalence filtering and DA testing operate on correctly paired samples. Incorrect sample-to-group mapping would silently corrupt differential abundance results across the 38 Hackathon datasets.

## When to use

Use when preparing a raw ASV/OTU count table and its metadata file for differential abundance analysis and you must verify that every sample name in the count table maps to exactly one grouping in the metadata, preserving sample order.

## When NOT to use

- The metadata file has more than two groupings — run_all_tools.sh only supports two-column metadata with maximum 2 groupings
- Filtering would depend on the test statistic (e.g. filtering significant taxa post-hoc) — the prevalence filter must be applied independently of any test statistic
- You need multi-group or covariate-rich metadata handling — this skill only covers two-column two-group design tables

## Inputs

- Raw 16S rRNA ASV abundance table (tsv, from the figshare 16S rRNA Microbiome Datasets deposit)
- Two-column metadata tsv (first column: sample names; second column: sample groupings, maximum 2 groupings)

## Outputs

- Prevalence-filtered abundance table (tsv, same structure as input, samples ordered as in metadata)
- phyloseq object pairing count table with sample_data metadata for downstream DA testing

## How to apply

Load the raw ASV count table into R with phyloseq, pairing the otu table with the two-column metadata tsv via sample_data so sample names resolve against the grouping column. Keep the sample order identical to the metadata file throughout, as required before applying the 10% prevalence filter (run_all_tools.sh -F 0.1) with phyloseq filter_taxa; the filter must be applied independently of any test statistic. The metadata file must be a two-column tsv with sample names in the first column and groupings in the second (maximum 2 groupings), matching run_all_tools.sh's -G argument. After filtering, write the filtered abundance table in the same tsv structure as the input, verifying that every sample present in the metadata grouping column is retained in the output.

## Related tools

- **phyloseq** (Loads the ASV count table and pairs it with the two-column metadata tsv via sample_data; filter_taxa applies the prevalence filter after sample alignment)
- **run_all_tools.sh** (Consumes the metadata file via -G and the ASV table via -A, enforcing the two-column tsv metadata format and applying the -F 0.1 prevalence filter on matched samples) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_0.1_filt_datasets.sh** (Runs all DA tools on the 38 prevalence-filtered datasets produced after metadata-matched filtering) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Every sample present in the metadata grouping column is retained in the filtered output table (no sample loss after matching)
- Sample order in the filtered abundance table is identical to the order in the metadata file
- Every retained ASV appears in at least 10% of samples (prevalence >= 0.1)
- Each of the 38 output files exists and retains the same tsv structure as its input count table
- The metadata file passes the format check: two-column tsv, sample names in column 1, groupings in column 2, maximum 2 groupings

## Limitations

- Metadata is restricted to a two-column tsv with sample names first and at most 2 groupings; covariates, repeated measures, or multi-group designs are unsupported
- The 10% prevalence threshold was arbitrarily selected; the authors note more work is needed to estimate an optimal cut-off
- Whether rarefying (the -D rarefaction depth, e.g. 2000) is advisable for other analyses such as diversity metrics is beyond the scope of this work

## Evidence

- [readme] Note that the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings): "Note that the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings)"
- [readme] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript): "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [methods] Remove any ASV with prevalence below the 0.1 threshold (the -F 0.1 setting of run_all_tools.sh, i.e. a 10% prevalence filter) using phyloseq filter_taxa, keeping the sample order identical to the metadata file: [evidence span withheld — anchor status: not_found]
- [methods] run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
