---
name: pcoa-ordination-of-tool-agreement
description: Use when you have significant-feature call sets from multiple DA tools
  run on the same datasets and need to quantify and visualize which tools agree, e.g.,
  to build a consensus approach or compare filtering regimes.
when_to_use_negative:
- Your goal is to identify which taxa differ between sample groups in a single dataset,
  not to benchmark tool agreement — the article's own question ('which taxa significantly
  differ...?') may be too simple and is a different task
- Input is a raw count/ASV table rather than per-tool significant-ASV output tables;
  run_all_tools.sh must first be executed on the dataset(s)
- Metadata has more than two groupings — run_all_tools.sh specifies a maximum of 2
  groupings per dataset
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_2269
tools:
- name: parallelDist
  role: Computes the pairwise Jaccard distances between the significant-ASV sets of
    every tool pair (Version 0.2.4)
- name: vegan
  role: Performs principal coordinates analysis on the mean Jaccard distance matrix
    (Version 2.5.6)
- name: ggplot2
  role: Plots the PCoA ordination labeling tools by clustering behavior (Version 3.3.0)
- name: dplyr
  role: Pools per-tool significant ASV calls and builds the tool-tally table (Version
    0.8.5)
- name: run_all_tools.sh
  role: Pipeline script that generates the per-dataset significant-ASV outputs from
    the 14 DA tools; filtering is enabled with -D 2000 -F 0.1
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:807f5f641b9853e7237e803c2d0d08ffdb716d74e3eab099fb3a0f9329a23eec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dplyr
  - ggplot2
  - parallelDist
  - run_all_tools.sh
  - vegan
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pcoa-ordination-of-tool-agreement

## Summary

Skill for benchmarking cross-tool agreement in differential abundance (DA) analysis: pairwise Jaccard distances are computed between the significant-ASV sets produced by each of 14 DA tools across 38 16S rRNA gene datasets, then visualized with principal coordinates analysis (PCoA) to reveal clustering of tools (e.g., edgeR/LEfSe grouping).

## When to use

Use when you have significant-feature call sets from multiple DA tools run on the same datasets and need to quantify and visualize which tools agree, e.g., to build a consensus approach or compare filtering regimes.

## When NOT to use

- Your goal is to identify which taxa differ between sample groups in a single dataset, not to benchmark tool agreement — the article's own question ('which taxa significantly differ...?') may be too simple and is a different task
- Input is a raw count/ASV table rather than per-tool significant-ASV output tables; run_all_tools.sh must first be executed on the dataset(s)
- Metadata has more than two groupings — run_all_tools.sh specifies a maximum of 2 groupings per dataset

## Inputs

- Per-dataset, per-tool significant ASV result tables from run_all_tools.sh (filtered and unfiltered runs of the 38 16S rRNA datasets)
- ASV table (ASV_TABLE.tsv) and two-column metadata TSV with sample names and maximum 2 groupings used to generate those runs

## Outputs

- consensus_asv_tool_counts.tsv: table tallying how many of the 14 tools called each ASV significantly DA, per filtering regime
- Mean Jaccard distance matrix (14x14 tool-by-tool, averaged over 38 datasets)
- fig3_pcoa_tool_jaccard.png: PCoA ordination figure of tool clustering (filtered vs. unfiltered variants)

## How to apply

1) Load the per-dataset, per-tool significant ASV result tables produced by run_all_tools.sh, both for the 38 filtered datasets (run_0.1_filt_datasets.sh with -F 0.1, i.e., a 10% prevalence filter) and the 38 unfiltered datasets (run_no_filt_datasets.sh). 2) Pool the significant ASV calls per tool and tally, for each ASV, how many of the 14 tools called it significantly DA, separately for filtered and unfiltered runs. 3) For each dataset, compute the pairwise Jaccard distance between the significant-ASV sets of every tool pair using the R parallelDist package (Version: 0.2.4), then average the distances across the 38 datasets. 4) Run principal coordinates analysis on the mean Jaccard distance matrix using the vegan package (Version: 2.5.6) in R 3.6.3. 5) Plot the ordination labeling tools by clustering behavior (e.g., edgeR/LEfSe grouping) with ggplot2 (Version: 3.3.0) to reproduce the tool-agreement figure; run both filtered and unfiltered variants since the overlap pattern changes with prevalence filtering.

## Related tools

- **parallelDist** (Computes the pairwise Jaccard distances between the significant-ASV sets of every tool pair (Version 0.2.4))
- **vegan** (Performs principal coordinates analysis on the mean Jaccard distance matrix (Version 2.5.6))
- **ggplot2** (Plots the PCoA ordination labeling tools by clustering behavior (Version 3.3.0))
- **dplyr** (Pools per-tool significant ASV calls and builds the tool-tally table (Version 0.8.5))
- **run_all_tools.sh** (Pipeline script that generates the per-dataset significant-ASV outputs from the 14 DA tools; filtering is enabled with -D 2000 -F 0.1) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods/wiki/Run-multiple-DA-tools-on-datasets-using-Docker-Image

## Examples

```
library(parallelDist); D <- parallelDist(binary_pa_matrix, method="jaccard"); ord <- vegan::cmdscale(D, k=2)
```

## Evaluation signals

- Filtered-data consensus distribution matches the reported pattern: proportion of significant ASVs identified by more than 12 tools is higher in prevalence-filtered data (mean 38.6%, SD 15.8%) than unfiltered (mean 17.3%, SD 22.1%)
- The PCoA of mean Jaccard distances reproduces the Fig. 3 clustering structure, e.g., edgeR/LEfSe grouping of similar tools
- The distance matrix is 14x14 (one row/column per DA tool) with off-diagonal Jaccard distances in [0, 1] and zero diagonal
- Ordination is computed separately for filtered and unfiltered runs and the two regimes are clearly separated in the summary statistics
- Tally table covers ASVs from all 38 datasets per regime with no missing tool columns

## Limitations

- The consensus intersection of tool outputs is not guaranteed to be more reliable; the tools may simply be picking up the same noise as significant
- The 10% prevalence filter cut-off was arbitrarily selected; the optimal cut-off is unknown
- The mean Jaccard distance averaging assumes the 38 datasets are comparable, but not all environments and datasets are appropriate for this comparison

## Evidence

- [abstract] we compare the performance of 14 differential abundance testing methods on 38 16S rRNA gene datasets with two sample groups: "we compare the performance of 14 differential abundance testing methods on 38 16S rRNA gene datasets with two sample groups"
- [abstract] these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
- [discussion] we recommend that users employ several methods and focus on significant features identified by most tools: "we recommend that users employ several methods and focus on significant features identified by most tools"
- [readme] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [readme] run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
