---
name: tsv-metadata-table-handling
description: Use when an unfiltered ASV count table (TSV) plus a two-column sample-group
  metadata TSV must be turned into depth-normalized inputs for run_all_tools.sh, i.e.,
  when samples have unequal read depths and a common rarefaction depth must be chosen
  and enforced.
when_to_use_negative:
- The count table is already rarefied or sample depths are equal — re-subsampling
  would discard data without benefit.
- The metadata file has more than two columns or more than two groupings; run_all_tools.sh
  expects a two-column TSV with maximum 2 groupings.
- The downstream analysis is one for which rarefying is out of scope (e.g. the article
  explicitly leaves open whether rarefying is advisable prior to computing diversity
  metrics).
tools:
- name: GUniFrac
  role: R package (Version 1.1) used to perform rarefaction, subsampling counts from
    each retained sample without replacement down to the common depth D
- name: run_all_tools.sh
  role: Key pipeline script that ingests the ASV table and metadata (-A, -G), applies
    rarefaction depth (-D) and prevalence filtering (-F), and writes the rarefied
    table (-R)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:ba66d965cff1744319f02bf4c6560184519dd486cdd6b2c4a94b52fc3db26717
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - GUniFrac
  - run_all_tools.sh
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# tsv-metadata-table-handling

## Summary

Prepares TSV-formatted ASV count tables and two-column metadata tables for the 14-tool differential abundance pipeline, including subsampling (rarefying) every sample to a common per-sample read depth (D = 2000 in the manuscript analysis) so that all samples have equal library size before DA testing.

## When to use

Use when an unfiltered ASV count table (TSV) plus a two-column sample-group metadata TSV must be turned into depth-normalized inputs for run_all_tools.sh, i.e., when samples have unequal read depths and a common rarefaction depth must be chosen and enforced.

## When NOT to use

- The count table is already rarefied or sample depths are equal — re-subsampling would discard data without benefit.
- The metadata file has more than two columns or more than two groupings; run_all_tools.sh expects a two-column TSV with maximum 2 groupings.
- The downstream analysis is one for which rarefying is out of scope (e.g. the article explicitly leaves open whether rarefying is advisable prior to computing diversity metrics).

## Inputs

- Unfiltered ASV count table as TSV (ASV_TABLE.tsv), samples as columns/features as rows
- Two-column metadata TSV (METADATA_FILE.tsv): column 1 sample names, column 2 sample groupings (maximum 2 groupings)
- Rarefaction depth parameter (D; e.g. 2000) and optional prevalence filter parameter (F; e.g. 0.1)

## Outputs

- Rarefied count table RARIFIED_ASV_TABLE.tsv with every retained sample at exactly D total reads and below-threshold samples removed
- Retained-sample subset of the metadata consistent with the rarefied table

## How to apply

Load the unfiltered ASV count table (ASV_TABLE.tsv) and a two-column metadata TSV where column 1 holds sample names and column 2 holds sample groupings (maximum 2 groupings). Compute each sample's total read depth and discard samples below the rarefaction depth passed via the -D flag (D = 2000 in the manuscript, invoked as 'run_all_tools.sh ... -D 2000 -F 0.1'). Apply rarefaction with the GUniFrac R package (Version 1.1), subsampling counts from each retained sample without replacement down to the common depth so all samples have equal library size. Write the result to RARIFIED_ASV_TABLE.tsv, which run_all_tools.sh consumes via the -R flag; the same invocation can also apply a 10% prevalence filter via -F 0.1, a step the authors suggest performing prior to DA testing. Rarefying is a deliberately debated preprocessing choice — the article frames it as one of the contested areas of DA practice — so record whether tools consumed rarefied or un-rarefied counts when interpreting results.

## Related tools

- **GUniFrac** (R package (Version 1.1) used to perform rarefaction, subsampling counts from each retained sample without replacement down to the common depth D)
- **run_all_tools.sh** (Key pipeline script that ingests the ASV table and metadata (-A, -G), applies rarefaction depth (-D) and prevalence filtering (-F), and writes the rarefied table (-R)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- RARIFIED_ASV_TABLE.tsv exists and every retained sample's total read count equals the chosen rarefaction depth D (e.g. column sums all equal 2000).
- Samples whose original depth was below the -D threshold are absent from RARIFIED_ASV_TABLE.tsv.
- Metadata file is a two-column TSV whose first-column sample names match the samples in the (rarefied) count table and whose second column contains at most 2 groupings.
- The prevalence-filtered run (-F 0.1) retains only features present in at least 10% of samples.
- Downstream DA output distinguishes tools that consumed the rarefied table (e.g. LEfSe and MaAsLin2 (rare) in Table 1) from those consuming un-rarefied counts.

## Limitations

- Rarefaction is heavily criticized for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests; results depend on data pre-processing.
- The manuscript's thresholds were arbitrary choices — D = 2000 was the rarefaction depth and 10% prevalence (F = 0.1) was arbitrarily selected rather than optimized.
- Whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work.

## Evidence

- [readme] run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] metadata file format constraint: "Note that the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings)"
- [methods] -D and -F parameter semantics: "-D represents the rarefaction depth"
- [intro] rarefaction debate motivating the preprocessing step: "One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples"
