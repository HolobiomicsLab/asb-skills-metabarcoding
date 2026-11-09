---
name: microbiome-library-size-normalization
description: Use when preparing 16S rRNA gene ASV/OTU count tables for differential
  abundance tools that require rarefied input, or when reproducing the rarefaction-vs-no-rarefaction
  arm of a DA method comparison.
when_to_use_negative:
- The downstream DA tool is configured to take un-rarefied count data — Table 1 specifies
  only LEfSe and MaAsLin2 (rare) took rarefied count tables as input
- You are deciding whether to rarefy prior to computing diversity metrics — the paper
  states that 'whether rarefying is advisable for other analyses, such as prior to
  computing diversity metrics, is beyond the scope of this work'
- The count table has already been rarefied to equal library size (re-rarefying adds
  no information)
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: GUniFrac
  role: R package (Version 1.1) used to subsample counts from each retained sample
    without replacement down to the common rarefaction depth
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_all_tools.sh
  role: Pipeline script that accepts the rarefaction depth via -D and the rarefied
    table via -R, applying filtering before DA tools run
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:2231438116e5c2fe5a071101f3363b03c0cc97b862b45b7c7dba94b580f84435
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - GUniFrac
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# microbiome-library-size-normalization

## Summary

Rarefies (subsamples) a 16S rRNA ASV count table to a common per-sample library size so that differential abundance tools configured for rarefied input (e.g., LEfSe and MaAsLin2 (rare) in the 14-tool comparison) receive equal-depth count data.

## When to use

Use when preparing 16S rRNA gene ASV/OTU count tables for differential abundance tools that require rarefied input, or when reproducing the rarefaction-vs-no-rarefaction arm of a DA method comparison.

## When NOT to use

- The downstream DA tool is configured to take un-rarefied count data — Table 1 specifies only LEfSe and MaAsLin2 (rare) took rarefied count tables as input
- You are deciding whether to rarefy prior to computing diversity metrics — the paper states that 'whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work'
- The count table has already been rarefied to equal library size (re-rarefying adds no information)

## Inputs

- ASV_TABLE.tsv — unfiltered ASV count table
- METADATA_FILE.tsv — two-column TSV with sample names (column 1) and sample groupings (column 2, maximum 2 groupings)
- Rarefaction depth threshold D (e.g., 2000 reads)

## Outputs

- RARIFIED_ASV_TABLE.tsv — rarefied count table with all retained samples at the common read depth, samples below the threshold absent

## How to apply

Load the unfiltered ASV count table (ASV_TABLE.tsv) and a two-column metadata TSV (sample names in column 1, sample groupings in column 2, maximum 2 groupings). Compute each sample's total read depth and discard samples whose depth falls below the rarefaction depth passed via the -D parameter of run_all_tools.sh (D = 2000 in the manuscript analysis, invoked as 'run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1'). Apply rarefaction with the GUniFrac R package (Version 1.1), subsampling counts from each retained sample without replacement down to the common depth so all samples have equal library size. Write the result to RARIFIED_ASV_TABLE.tsv, which is supplied to run_all_tools.sh via the -R flag. Rationale: the paper treats rarefaction as one of the debated pre-processing choices that changes DA results, so the rarefied table is kept as a separate input so only tools configured for it (LEfSe and MaAsLin2 (rare)) consume it.

## Related tools

- **GUniFrac** (R package (Version 1.1) used to subsample counts from each retained sample without replacement down to the common rarefaction depth) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_all_tools.sh** (Pipeline script that accepts the rarefaction depth via -D and the rarefied table via -R, applying filtering before DA tools run) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -R RARIFIED_ASV_TABLE.tsv
```

## Evaluation signals

- Every retained sample in RARIFIED_ASV_TABLE.tsv has a total read count exactly equal to the chosen rarefaction depth
- Samples whose original depth was below the -D threshold (e.g., 2000) are absent from RARIFIED_ASV_TABLE.tsv
- The output file exists and its sample identifiers match the retained rows of the two-column metadata TSV
- Rerunning the rarefaction on the same ASV_TABLE.tsv and metadata with the same D reproduces equal-depth column sums across samples

## Limitations

- Rarefaction has been heavily criticized for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests
- Samples below the rarefaction depth are discarded entirely, so very shallow samples cannot be rescued
- Whether rarefying is advisable for analyses other than DA testing (e.g., diversity metrics) is beyond the scope of this work
- DA results depend on data pre-processing, so the rarefied and un-rarefied tables must be routed to the correct tools per Table 1

## Evidence

- [intro] One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples3. This approach has been heavily criticized: "One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples3. This approach has been heavily criticized"
- [methods] run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [methods] -D represents the rarefaction depth: "-D represents the rarefaction depth"
- [readme] "GUniFrac" Version: 1.1: "R version: 3.6.3
  * A list of required R packages:
    * "GUniFrac" Version: 1.1"
- [readme] Note that the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings): "Note that the metadatafile should be a two column tsv table with the first column having the sample names and the second column having the sample groupings.(maximum 2 groupings)"
