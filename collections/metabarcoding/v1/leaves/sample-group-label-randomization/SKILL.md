---
name: sample-group-label-randomization
description: 'Use when benchmarking DA tools'' false-positive control on 16S rRNA
  ASV count tables: you need to know which tools (e.g., edgeR, LEfSe) call spurious
  significant ASVs when group labels carry no real signal.'
when_to_use_negative:
- When you need true differential abundance results on real case/control comparisons
  — randomized labels guarantee no group signal by construction
- When estimating practical FDR for real sample groupings, since the randomized approach
  assumes no systematic abundance differences that real groups may contain
- When data has not been preprocessed (e.g., no 10% prevalence filter applied) and
  you intend to compare against the paper's filtered-data benchmarks
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: edgeR
  role: DA tool benchmarked; flagged as the main false-positive outlier (mean 0.69–27.9%
    significant ASVs on filtered data)
- name: LEfSe
  role: DA tool benchmarked; second outlier (mean 3.4–5.1% significant ASVs on filtered
    data)
- name: ANCOM
  role: DA tool benchmarked; excluded from the 90-replicate unfiltered FP extension
    run
  repo: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- name: ALDEx2
  role: DA tool benchmarked; yields 0% on unfiltered rarefied data and excluded from
    the 90-replicate extension run
- name: corncob
  role: DA tool benchmarked; excluded from the 90-replicate unfiltered FP extension
    run
- name: run_all_tools.sh
  role: Pipeline script that runs all DA tools on a permuted-label dataset (with -D
    2000 -F 0.1 for filtered runs)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:773a2d6fdfea06119b366f7dbda9921e7378bb8d9dbcc91de019688ed9a7bb38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
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
  - run_all_tools.sh
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# sample-group-label-randomization

## Summary

Estimates the false-positive rate of microbiome differential abundance (DA) tools by randomly reassigning case/control labels among samples within the most frequently sampled group of a dataset, so that no true group differences exist, and measuring the percent of ASVs each tool calls significant.

## When to use

Use when benchmarking DA tools' false-positive control on 16S rRNA ASV count tables: you need to know which tools (e.g., edgeR, LEfSe) call spurious significant ASVs when group labels carry no real signal.

## When NOT to use

- When you need true differential abundance results on real case/control comparisons — randomized labels guarantee no group signal by construction
- When estimating practical FDR for real sample groupings, since the randomized approach assumes no systematic abundance differences that real groups may contain
- When data has not been preprocessed (e.g., no 10% prevalence filter applied) and you intend to compare against the paper's filtered-data benchmarks

## Inputs

- Raw 16S rRNA ASV count tables (ASV_TABLE.tsv) from the figshare 16S rRNA Microbiome Datasets deposit
- Two-column TSV metadata files (sample names and sample groupings, maximum 2 groupings)
- Rarefied ASV table (RARIFIED_ASV_TABLE.tsv) for no-filtering runs

## Outputs

- Per-tool, per-replicate percentage of ASVs called significantly differentially abundant under randomized labels
- Aggregated results table matching the Fig. 4 layout, covering both prevalence-filtered and unfiltered conditions

## How to apply

Download the raw 16S rRNA ASV count tables and two-column metadata (sample IDs and groupings) for the selected datasets from the figshare deposit (https://figshare.com/articles/dataset/16S_rRNA_Microbiome_Datasets/14531724). For each dataset, identify the most frequent sample group and randomly permute the case/control labels among its samples; for the unfiltered extension, run 90 replicates via run_FP_unfilt_EXT.sh, which applies to all tools except ANCOM, ALDEx2, and corncob. Run each DA tool on the permuted-label data with run_all_FP_datasets_filt.sh (10% prevalence filter F=0.1, rarefaction depth D=2000) or run_all_FP_datasets_unfilt.sh, invoking run_all_tools.sh per dataset. For every tool and replicate, compute the percent of ASVs called significantly differentially abundant under the randomized labels, then aggregate per-tool percentages across replicates and datasets into a table matching Fig. 4's layout for both filtered and unfiltered conditions.

## Related tools

- **edgeR** (DA tool benchmarked; flagged as the main false-positive outlier (mean 0.69–27.9% significant ASVs on filtered data))
- **LEfSe** (DA tool benchmarked; second outlier (mean 3.4–5.1% significant ASVs on filtered data))
- **ANCOM** (DA tool benchmarked; excluded from the 90-replicate unfiltered FP extension run) — https://github.com/FrederickHuangLin/ANCOM-Code-Archive
- **ALDEx2** (DA tool benchmarked; yields 0% on unfiltered rarefied data and excluded from the 90-replicate extension run)
- **corncob** (DA tool benchmarked; excluded from the 90-replicate unfiltered FP extension run)
- **run_all_tools.sh** (Pipeline script that runs all DA tools on a permuted-label dataset (with -D 2000 -F 0.1 for filtered runs)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- edgeR on 10% prevalence-filtered data should yield a mean of 0.69–27.9% significant ASVs across datasets; flag any tool/dataset combination outside reported bounds
- LEfSe on filtered data should yield a mean of 3.4–5.1% significant ASVs
- ALDEx2 and the t-test on rarefied data should consistently yield 0% significant ASVs on unfiltered data
- The unfiltered extension run should contain 90 replicates per dataset for all tools except ANCOM, ALDEx2, and corncob
- A tool with good FDR control should call close to 0% of ASVs significant, since randomized labels create no expected group differences

## Limitations

- The randomized approach for estimating FDR is not a perfect representation of real data; real sample groupings will likely contain some systematic differences in microbial abundances, so few significant ASVs here is not proof of low practical FDR
- ANCOM, ALDEx2, and corncob are excluded from the 90-replicate unfiltered extension, limiting replicate depth for those tools
- Reasons why some methods have much higher significance rates than other tools remain unclear

## Evidence

- [readme] FP filtered script: "run_all_FP_datasets_filt.sh --> runs the false postive analysis for filtered datasets"
- [readme] FP extension replicates: "run_FP_unfilt_EXT.sh --> runs an additional 90 replicates for the unfilt FP analysis on all tools except for ANCOM, ALDEx2, and corncob."
- [readme] filtering parameters: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [discussion] FDR caveat: "our randomized approach for estimating FDR is not a perfect representation of real data; that is, real sample groupings will likely contain some systematic"
- [abstract] scope of benchmark: "we compare the performance of 14 differential abundance testing methods on 38 16S rRNA gene datasets with two sample groups"
