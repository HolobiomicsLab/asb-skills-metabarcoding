---
name: amplicon-count-table-rarefaction
description: Use when preparing a two-column-metadata 16S rRNA ASV count table for
  differential abundance tools that require equal library sizes (rarefied input),
  so all samples share the same total read count after subsampling.
when_to_use_negative:
- The downstream DA tool expects un-rarefied count data — most of the 14 tools in
  the comparison took un-rarefied counts; only LEfSe and MaAsLin2 (rare) were configured
  for rarefied input.
- The goal is an analysis other than DA testing — the article states that 'whether
  rarefying is advisable for other analyses, such as prior to computing diversity
  metrics, is beyond the scope of this work'.
- The input table is already rarefied or all samples already share equal library sizes;
  re-rarefying would discard reads unnecessarily.
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: GUniFrac
  role: R package (Version 1.1) used to subsample counts from each retained sample
    without replacement down to the common rarefaction depth
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_all_tools.sh
  role: Pipeline script that accepts -D (rarefaction depth) and -R (rarefied ASV table)
    parameters and supplies the rarefied table to downstream DA tools
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:df3acfeff098dc1c6178735791327d949e236a0795a13d0c7627d4779cc91c9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - GUniFrac
  - run_all_tools.sh
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# amplicon-count-table-rarefaction

## Summary

Rarefies an unfiltered 16S rRNA ASV count table to a common per-sample read depth (D = 2000 in the manuscript analysis) using the GUniFrac R package, after discarding samples below the depth threshold, producing a rarefied table for DA tools configured to consume it (e.g., LEfSe and MaAsLin2 (rare)).

## When to use

Use when preparing a two-column-metadata 16S rRNA ASV count table for differential abundance tools that require equal library sizes (rarefied input), so all samples share the same total read count after subsampling.

## When NOT to use

- The downstream DA tool expects un-rarefied count data — most of the 14 tools in the comparison took un-rarefied counts; only LEfSe and MaAsLin2 (rare) were configured for rarefied input.
- The goal is an analysis other than DA testing — the article states that 'whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work'.
- The input table is already rarefied or all samples already share equal library sizes; re-rarefying would discard reads unnecessarily.

## Inputs

- Unfiltered ASV count table (ASV_TABLE.tsv) with samples as columns and ASVs as rows
- Two-column metadata TSV (METADATA_FILE.tsv): column 1 sample names, column 2 sample groupings (maximum 2 groupings)
- Rarefaction depth parameter D (e.g., 2000)
- Optional prevalence filter parameter F (e.g., 0.1 for a 10% prevalence filter)

## Outputs

- Rarefied count table (RARIFIED_ASV_TABLE.tsv) in which every retained sample totals exactly the chosen rarefaction depth and samples under the depth threshold are absent

## How to apply

Load the unfiltered ASV count table (ASV_TABLE.tsv) and a two-column metadata TSV (sample names in column 1, groupings in column 2, maximum 2 groupings). Compute each sample's total read depth and discard samples below the rarefaction depth passed via -D (D = 2000 in 'run_all_tools.sh ... -D 2000 -F 0.1'), optionally combined with the prevalence filter -F (0.1 = 10% prevalence filter in the manuscript). Apply rarefaction with the GUniFrac R package (Version 1.1), subsampling counts from each retained sample without replacement down to the common depth. Write the result to RARIFIED_ASV_TABLE.tsv, which run_all_tools.sh supplies to downstream tools via the -R flag; in the 14-tool comparison only LEfSe (TSS normalization, no transformation) and MaAsLin2 (rare) (TSS normalization, AST transformation) consumed this rarefied table, while other tools used un-rarefied data.

## Related tools

- **GUniFrac** (R package (Version 1.1) used to subsample counts from each retained sample without replacement down to the common rarefaction depth) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_all_tools.sh** (Pipeline script that accepts -D (rarefaction depth) and -R (rarefied ASV table) parameters and supplies the rarefied table to downstream DA tools) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- RARIFIED_ASV_TABLE.tsv exists after the run.
- Every retained sample in RARIFIED_ASV_TABLE.tsv has a total read count exactly equal to the chosen rarefaction depth (e.g., column sums all equal D = 2000).
- Samples whose original depth was below the -D threshold are absent from the rarefied table.
- Sample IDs match the two-column metadata file, with at most 2 groupings represented.
- The rarefied table is passed via the -R flag to run_all_tools.sh and consumed only by the tools configured for rarefied input (LEfSe, MaAsLin2 (rare)), not the un-rarefied-input tools.

## Limitations

- Rarefaction is heavily criticized in the differential abundance literature for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests — the article lists it as a key area of disagreement.
- The rarefaction depth (D = 2000) and prevalence filter (F = 0.1) were arbitrarily selected for the manuscript analysis; no optimal cut-off has been established.
- Whether rarefying is advisable for analyses beyond DA testing, such as computing diversity metrics, is beyond the scope of this work.
- Metadata must be a two-column TSV with a maximum of 2 groupings; datasets violating this format are not supported by the pipeline invocation shown.

## Evidence

- [intro] One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples: "One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples3. This approach has been heavily criticized"
- [readme] run_all_tools.sh filtering invocation with depth and prevalence parameters: "To run this script on a single dataset with filtering you can run the following command: 
  
  run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
  "
- [readme] Meaning of the -D parameter: "* -D represents the rarefaction depth"
- [readme] Meaning of the -F prevalence filter parameter: "* -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [methods] Unfiltered run supplying the rarefied table via -R: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -R RARIFIED_ASV_TABLE.tsv"
- [readme] GUniFrac required R package version: "* "GUniFrac" Version: 1.1"
