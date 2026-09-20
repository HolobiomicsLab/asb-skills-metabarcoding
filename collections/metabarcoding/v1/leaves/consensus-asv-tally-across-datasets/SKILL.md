---
name: consensus-asv-tally-across-datasets
description: Use when you have significant-ASV result tables from multiple DA tools
  (14 tools in the Nearing et al. 2022 pipeline) run over many datasets (38 filtered
  + 38 unfiltered) and need to measure how much the tools' significant-feature sets
  overlap and whether prevalence filtering increases agreement.
when_to_use_negative:
- You only ran a single DA tool on a single dataset — there are no tool pairs to compute
  Jaccard overlap or consensus tallies across
- You need absolute truth about which ASVs are truly differential — this skill measures
  inter-tool agreement only; the paper warns the intersection of tool outputs may
  be the same noise
- The ASV table has not been preprocessed with the same prevalence filter (-F 0.1)
  across the compared runs — filtered/unfiltered consensus proportions are not comparable
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: parallelDist
  role: 'Computes the pairwise Jaccard distance matrix between the significant-ASV
    sets of every tool pair (Version: 0.2.4)'
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: vegan
  role: 'Runs principal coordinates analysis on the mean Jaccard distance matrix (Version:
    2.5.6) in R 3.6.3'
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: ggplot2
  role: 'Plots the PCoA ordination labeling tools by clustering behavior to match
    Fig. 3 (Version: 3.3.0)'
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: dplyr
  role: Pools significant-ASV calls per tool and builds the per-ASV tool-count tally
    table
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:4668347ab18f495296692d3cdd75002fdb3542dcbffab89a0ef7ab5835ffc579
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - dplyr
  - ggplot2
  - parallelDist
  - vegan
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# consensus-asv-tally-across-datasets

## Summary

Quantify cross-tool consensus by tallying, per ASV, how many of the 14 differential abundance tools called it significantly DA, then compare consensus structure between 10% prevalence-filtered and unfiltered 16S rRNA runs (filtered data showed 38.6% vs 17.3% mean of ASVs called by >12 tools). Supports the paper's recommendation to focus on features identified by most tools.

## When to use

Use when you have significant-ASV result tables from multiple DA tools (14 tools in the Nearing et al. 2022 pipeline) run over many datasets (38 filtered + 38 unfiltered) and need to measure how much the tools' significant-feature sets overlap and whether prevalence filtering increases agreement.

## When NOT to use

- You only ran a single DA tool on a single dataset — there are no tool pairs to compute Jaccard overlap or consensus tallies across
- You need absolute truth about which ASVs are truly differential — this skill measures inter-tool agreement only; the paper warns the intersection of tool outputs may be the same noise
- The ASV table has not been preprocessed with the same prevalence filter (-F 0.1) across the compared runs — filtered/unfiltered consensus proportions are not comparable

## Inputs

- Per-dataset, per-tool significant ASV result tables from run_all_tools.sh (38 filtered datasets via run_0.1_filt_datasets.sh; 38 unfiltered via run_no_filt_datasets.sh)
- Mapfiles from Misc_datafiles to reconcile working vs clean study and tool names

## Outputs

- consensus_asv_tool_counts.tsv: per-ASV tally (0–14) of tools calling it significantly DA, for filtered and unfiltered runs
- fig3_pcoa_tool_jaccard.png: PCoA ordination of tools based on mean pairwise Jaccard distance across the 38 datasets

## How to apply

Load the per-dataset, per-tool significant ASV tables produced by run_all_tools.sh from the filtered runs (run_0.1_filt_datasets.sh, with -D 2000 -F 0.1) and unfiltered runs (run_no_filt_datasets.sh). Pool calls per tool and tally for each ASV how many of the 14 tools flagged it significantly DA, separately for filtered and unfiltered data. Compute pairwise Jaccard distances between tool-specific significant-ASV sets with the R parallelDist package (v0.2.4), average across the 38 datasets, and run principal coordinates analysis with vegan (v2.5.6) in R 3.6.3. Plot the ordination with ggplot2 (v3.3.0) labeling tools by clustering behavior (e.g. edgeR/LEfSe grouping) to match Fig. 3, and write the per-ASV tool-count table (consensus_asv_tool_counts.tsv). The rationale is the paper's finding that filtering increases cross-tool agreement and that a consensus of tools is the recommended way to pick robust features.

## Related tools

- **parallelDist** (Computes the pairwise Jaccard distance matrix between the significant-ASV sets of every tool pair (Version: 0.2.4)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **vegan** (Runs principal coordinates analysis on the mean Jaccard distance matrix (Version: 2.5.6) in R 3.6.3) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **ggplot2** (Plots the PCoA ordination labeling tools by clustering behavior to match Fig. 3 (Version: 3.3.0)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **dplyr** (Pools significant-ASV calls per tool and builds the per-ASV tool-count tally table) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1
```

## Evaluation signals

- Every ASV tally falls within 0–14 tools and every tool's significant set is non-empty per dataset
- Filtered-data summary reproduces the reported pattern: mean proportion of ASVs called by >12 tools ≈ 38.6% (SD 15.8%) vs ≈ 17.3% (SD 22.1%) unfiltered — i.e. filtering increases cross-tool agreement
- The mean Jaccard distance matrix (averaged over the 38 datasets) is symmetric with zero diagonal before PCoA
- PCoA ordination reproduces the Fig. 3 grouping, e.g. edgeR and LEfSe clustering together, ALDEx2/ANCOM-II in their reported positions
- Ordination output (fig3_pcoa_tool_jaccard.png) and tally table (consensus_asv_tool_counts.tsv) exist and row counts match the number of ASVs in the pooled tables

## Limitations

- The consensus intersection of tool outputs is not guaranteed to be more reliable; the paper notes tools may simply be picking up the same noise as significant
- The 10% prevalence filter cut-off (-F 0.1) was arbitrarily selected; the optimal cut-off is unknown
- The random expectation for cross-study consistency was simplistic (assuming all genera equally likely to be significant by chance), which is invalid due to prevalence differences
- Results depend on data pre-processing, so filtered vs unfiltered consensus tallies are not directly comparable without stating the preprocessing used

## Evidence

- [discussion] we recommend that users employ several methods and focus on significant features identified by most tools, while keeping in mind the characteristics of the tools presented within this manuscript.: "we recommend that users employ several methods and focus on significant features identified by most tools, while keeping in mind the characteristics of the tools presented within this manuscript."
- [abstract] ALDEx2 and ANCOM-II produce the most consistent results across studies and agree best with the intersect of results from different approaches.: "ALDEx2 and ANCOM-II produce the most consistent results across studies and agree best with the intersect of results from different approaches."
- [readme] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [readme] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript): "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [discussion] there is no assurance that the intersection of the tool outputs is more reliable; it is possible that the tools are simply picking up the same noise as significant.: "there is no assurance that the intersection of the tool outputs is more reliable; it is possible that the tools are simply picking up the same noise as significant."
