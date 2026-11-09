---
name: cross-tool-jaccard-distance-computation
description: Use when you have per-dataset, per-tool significant-feature result tables
  and need to measure how much different DA tools agree (or disagree) on which ASVs
  are significantly differentially abundant, including under different prevalence-filtering
  regimes.
when_to_use_negative:
- The goal is cross-study consistency (same tool across datasets from the same environment,
  e.g. the obesity or diarrhea consistency analyses) rather than cross-tool agreement
  within a dataset
- Only one differential abundance tool has been run, so there are no per-tool significant-ASV
  sets to compare pairwise
- The question is absolute accuracy or false positive rate of a tool; that requires
  the randomized false-positive analysis (run_all_FP_datasets_filt.sh / run_all_FP_datasets_unfilt.sh),
  not Jaccard overlap
tools:
- name: parallelDist
  role: 'Computes the pairwise Jaccard distances between the significant-ASV sets
    of every tool pair within each dataset (Version: 0.2.4)'
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: vegan
  role: 'Runs principal coordinates analysis (PCoA) on the mean Jaccard distance matrix
    (Version: 2.5.6)'
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ggplot2
  role: 'Plots the PCoA ordination labeling tools by clustering behavior to match
    Fig. 3 (Version: 3.3.0)'
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: dplyr
  role: Data manipulation for pooling significant-ASV calls per tool and tallying
    how many tools called each ASV significant
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: run_all_tools.sh
  role: Key pipeline script that runs all 14 DA tools on a given dataset to generate
    the per-tool significant-ASV inputs
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:e2d885b0919cde03f8d6cbbf3710f368043e1e728743f70b63c45e5b07e39172
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - dplyr
  - ggplot2
  - parallelDist
  - run_all_tools.sh
  - vegan
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# cross-tool-jaccard-distance-computation

## Summary

Quantifies agreement between differential abundance tools by computing pairwise Jaccard distances between the significant-ASV sets each tool reports per dataset (here 14 tools on 38 filtered and 38 unfiltered 16S rRNA datasets), then ordinating the mean distance matrix by PCoA to reveal tool clustering behavior.

## When to use

Use when you have per-dataset, per-tool significant-feature result tables and need to measure how much different DA tools agree (or disagree) on which ASVs are significantly differentially abundant, including under different prevalence-filtering regimes.

## When NOT to use

- The goal is cross-study consistency (same tool across datasets from the same environment, e.g. the obesity or diarrhea consistency analyses) rather than cross-tool agreement within a dataset
- Only one differential abundance tool has been run, so there are no per-tool significant-ASV sets to compare pairwise
- The question is absolute accuracy or false positive rate of a tool; that requires the randomized false-positive analysis (run_all_FP_datasets_filt.sh / run_all_FP_datasets_unfilt.sh), not Jaccard overlap

## Inputs

- Per-dataset, per-tool significant ASV result tables from run_all_tools.sh (filtered: run_0.1_filt_datasets.sh; unfiltered: run_no_filt_datasets.sh)
- ASV table and two-column TSV metadata file (sample names and groupings, maximum 2 groupings)
- Prevalence-filtering parameter (e.g. -F 0.1 for 10%) and optional rarefaction depth (e.g. -D 2000)

## Outputs

- consensus_asv_tool_counts.tsv — per-ASV tally of how many of the 14 tools called each ASV significantly DA (filtered and unfiltered)
- Mean cross-tool Jaccard distance matrix (tools x tools, averaged over 38 datasets)
- fig3_pcoa_tool_jaccard.png — PCoA ordination of tools based on mean Jaccard distance, colored/labeled by clustering behavior

## How to apply

Load the significant-ASV result tables produced by run_all_tools.sh on the 38 filtered datasets (run_0.1_filt_datasets.sh with -F 0.1, i.e. a 10% prevalence filter) and the 38 unfiltered datasets (run_no_filt_datasets.sh). Pool the significant-ASV calls per tool and tally, for each ASV, how many of the 14 tools called it significantly DA, separately for filtered and unfiltered runs. For each dataset, compute the pairwise Jaccard distance between the significant-ASV sets of every tool pair using the R parallelDist package (Version: 0.2.4) in R 3.6.3, then average the distances across the 38 datasets. Run principal coordinates analysis on the mean Jaccard distance matrix with the vegan package (Version: 2.5.6) and plot the ordination labeling tools by clustering behavior (e.g. edgeR/LEfSe grouping) with ggplot2 (Version: 3.3.0) to reproduce the Fig. 3-style ordination. Comparing the filtered vs unfiltered tallies tests whether prevalence filtering increases cross-tool agreement.

## Related tools

- **parallelDist** (Computes the pairwise Jaccard distances between the significant-ASV sets of every tool pair within each dataset (Version: 0.2.4)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **vegan** (Runs principal coordinates analysis (PCoA) on the mean Jaccard distance matrix (Version: 2.5.6)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ggplot2** (Plots the PCoA ordination labeling tools by clustering behavior to match Fig. 3 (Version: 3.3.0)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **dplyr** (Data manipulation for pooling significant-ASV calls per tool and tallying how many tools called each ASV significant) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **run_all_tools.sh** (Key pipeline script that runs all 14 DA tools on a given dataset to generate the per-tool significant-ASV inputs) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
R> d <- parallelDist(sig_asv_matrix, method="binary"); pco <- cmdscale(d, k=2)  # sig_asv_matrix: 14 tools x ASVs from run_all_tools.sh outputs, averaged over 38 datasets
```

## Evaluation signals

- The mean Jaccard distance matrix must be symmetric with a zero diagonal (each tool's distance to itself is 0) and off-diagonal values in [0, 1]
- The tool-tally table (consensus_asv_tool_counts.tsv) should show counts between 0 and 14 tools per ASV, and the filtered-data summary should show a higher proportion of ASVs called by more than 12 tools than the unfiltered data (reported means: 38.6% filtered vs 17.3% unfiltered), demonstrating filtering increases cross-tool agreement
- The PCoA ordination should reproduce the known tool clustering behavior from Fig. 3, e.g. edgeR and LEfSe grouping together
- Averages must be taken across exactly 38 datasets per condition (filtered and unfiltered runs)
- Distributions should show most ASVs being called significant by either many tools or few tools, matching the paper's reported bimodal pattern

## Limitations

- Results depend on data pre-processing: the cross-tool overlap and ordination differ between filtered (-F 0.1) and unfiltered runs, so conclusions are conditional on the filtering regime
- The consensus intersection of tool outputs is not guaranteed to be more reliable; it is possible that the tools are simply picking up the same noise as significant
- It remains unclear why some methods have much higher significance rates than others, so differences in overlap may reflect uncharacterized tool-specific behavior rather than true agreement
- A direct simple recommendation of a specific tool could not be made from this analysis; the Jaccard/PCoA view mainly says which tools are more similar to others, not which is correct

## Evidence

- [abstract] we compare the performance of 14 differential abundance testing methods on 38 16S rRNA gene datasets with two sample groups: "we compare the performance of 14 differential abundance testing methods on 38 16S rRNA gene datasets with two sample groups"
- [abstract] results depend on data pre-processing: "these tools identified drastically different numbers and sets of significant ASVs, and that results depend on data pre-processing"
- [readme] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [readme] The key script is run_all_tools.sh for running all tools on a given dataset: "The key script is "run_all_tools.sh" for running all tools on a given dataset."
- [methods] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [readme] vegan Version 2.5.6 and parallelDist Version 0.2.4 listed among analysis packages: "* "vegan" Version: 2.5.6
  * "parallelDist" Version: 0.2.4"
- [discussion] performing prevalence filtering (e.g., at 10%) of features prior to DA testing: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off"
