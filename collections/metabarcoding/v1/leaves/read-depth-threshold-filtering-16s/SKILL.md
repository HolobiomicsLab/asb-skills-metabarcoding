---
name: read-depth-threshold-filtering-16s
description: Use when running the nearinj/Comparison_of_DA_microbiome_methods pipeline
  on an unfiltered ASV count table where differing per-sample read depths must be
  equalized before DA testing, or when a rarefied input table (-R) is required by
  downstream tools.
when_to_use_negative:
- The downstream analysis is computing diversity metrics or other uses of rarefying
  — the authors state that 'whether rarefying is advisable for other analyses, such
  as prior to computing diversity metrics, is beyond the scope of this work'.
- The DA tool explicitly requires absolute, un-rarefied count tables — e.g. the ANCOM
  documentation warns the input 'is the **absolute** abundance table, do not transform
  it to **relative** abundance table'.
- No samples fall below the depth threshold and no filtering is needed — run the no-filtering
  form 'run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -R
  RARIFIED_ASV_TABLE.tsv' instead.
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: GUniFrac
  role: R package (Version 1.1) used to subsample counts from each retained sample
    without replacement down to the common rarefaction depth
- name: run_all_tools.sh
  role: Key pipeline script that applies the -D rarefaction depth and -F prevalence
    filtering and distributes the rarefied table (via -R) to the 14 DA tools
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:a48d7aa241b77694afddf6da1e5930e75bdd8bb73bd32036f954148b0caa4aad
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - GUniFrac
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# read-depth-threshold-filtering-16s

## Summary

Filter 16S rRNA ASV count tables by a per-sample read-depth threshold (D = 2000 in the manuscript) and rarefy retained samples to that common depth with the GUniFrac R package, producing the rarefied table consumed by depth-sensitive differential abundance tools such as LEfSe and MaAsLin2 (rare).

## When to use

Use when running the nearinj/Comparison_of_DA_microbiome_methods pipeline on an unfiltered ASV count table where differing per-sample read depths must be equalized before DA testing, or when a rarefied input table (-R) is required by downstream tools.

## When NOT to use

- The downstream analysis is computing diversity metrics or other uses of rarefying — the authors state that 'whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work'.
- The DA tool explicitly requires absolute, un-rarefied count tables — e.g. the ANCOM documentation warns the input 'is the **absolute** abundance table, do not transform it to **relative** abundance table'.
- No samples fall below the depth threshold and no filtering is needed — run the no-filtering form 'run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -R RARIFIED_ASV_TABLE.tsv' instead.

## Inputs

- Unfiltered ASV count table TSV (ASV_TABLE.tsv)
- Two-column metadata TSV (column 1: sample names; column 2: sample groupings, maximum 2 groupings)
- Rarefaction depth threshold (-D, 2000 in the manuscript analysis)
- Rarefied count table path (-R, RARIFIED_ASV_TABLE.tsv) when running without filtering

## Outputs

- Rarefied ASV count table TSV (RARIFIED_ASV_TABLE.tsv) with equal library size per retained sample
- Tool outputs from run_all_tools.sh for depth-sensitive DA tools (e.g. LEfSe, MaAsLin2 (rare)) run on the rarefied table

## How to apply

Load the unfiltered ASV count table (ASV_TABLE.tsv) and the two-column metadata TSV (sample names in column 1, up to 2 sample groupings in column 2). Compute each sample's total read depth and discard samples below the rarefaction depth passed via the -D parameter (D = 2000 in the manuscript analysis, e.g. 'run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1'). Apply rarefaction with GUniFrac (Version 1.1), subsampling counts from each retained sample without replacement down to the common depth so all samples have equal library size. Write the result to RARIFIED_ASV_TABLE.tsv, which is supplied to run_all_tools.sh via the -R flag; Table 1 of the article specifies that LEfSe (TSS normalization, no transformation) and MaAsLin2 (rare) (TSS normalization, AST transformation) take rarefied count tables as input. The rationale is contested: rarefying corrects for differing read depths but has been heavily criticized for reducing statistical power, introducing biases, and yielding high false positive rates under standard tests.

## Related tools

- **GUniFrac** (R package (Version 1.1) used to subsample counts from each retained sample without replacement down to the common rarefaction depth)
- **run_all_tools.sh** (Key pipeline script that applies the -D rarefaction depth and -F prevalence filtering and distributes the rarefied table (via -R) to the 14 DA tools) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- RARIFIED_ASV_TABLE.tsv exists and the total read count of every retained sample equals the chosen rarefaction depth (D = 2000 in the manuscript analysis).
- Samples whose original depth fell below D are absent from RARIFIED_ASV_TABLE.tsv.
- Rarefaction was performed with GUniFrac Version 1.1 using subsampling without replacement, so all retained samples share equal library size.
- The rarefied table is wired via the -R flag so depth-sensitive tools (LEfSe and MaAsLin2 (rare), per Table 1) receive rarefied counts while other tools receive un-rarefied counts.
- Running the expected Human-ALL (MALL) dataset via run_all_tools.sh reproduces the expected output bundled in Misc_datafiles/Expected_MALL_output (~20 minutes on a single core).

## Limitations

- Rarefaction has been heavily criticized for reducing statistical power, introducing biases, and yielding high false positive rates with standard tests — the article lists it as a key area of disagreement in DA pre-processing.
- Whether rarefying is advisable for analyses other than DA testing (e.g., prior to computing diversity metrics) is beyond the scope of this work.
- The pipeline's results depend on data pre-processing; the depth threshold must be set explicitly via -D, and the manuscript's parameters were 'run_all_tools.sh ... -D 2000 -F 0.1'.

## Evidence

- [methods] run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [readme] -D represents the rarefaction depth: "-D represents the rarefaction depth"
- [intro] One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples: "One area of disagreement is whether read count tables should be rarefied (i.e., subsampled) to correct for differing read depths across samples3"
- [discussion] whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work: "whether rarefying is advisable for other analyses, such as prior to computing diversity metrics, is beyond the scope of this work3,4."
- [readme] The key script is "run_all_tools.sh" for running all tools on a given dataset.: "The key script is "run_all_tools.sh" for running all tools on a given dataset."
