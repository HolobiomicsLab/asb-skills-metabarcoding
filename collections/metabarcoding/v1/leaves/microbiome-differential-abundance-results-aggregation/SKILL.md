---
name: microbiome-differential-abundance-results-aggregation
description: Use when you have significant-ASV result tables from several DA tools
  (e.g., 14 tools) run on the same datasets and need to quantify overlap, consensus,
  or tool clustering, including effects of prevalence filtering.
when_to_use_negative:
- You are estimating false-positive rates on randomized/simulated data rather than
  aggregating real significant-ASV calls (the paper handles this with separate run_all_FP_datasets_filt.sh
  / run_all_FP_datasets_unfilt.sh scripts)
- You are assessing cross-study consistency of genera across datasets from the same
  environment (e.g., the diarrhea or obesity consistency analyses), which uses a different
  random-expectation comparison rather than tool-vs-tool Jaccard distances
- You only ran a single DA tool per dataset, so there is no cross-tool overlap to
  aggregate
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: parallelDist
  role: Computes pairwise Jaccard distances between the significant-ASV sets of every
    tool pair (v0.2.4)
- name: vegan
  role: Runs principal coordinates analysis on the mean tool-by-tool Jaccard distance
    matrix (v2.5.6, R 3.6.3)
- name: ggplot2
  role: Plots the PCoA ordination labeling tools by clustering behavior (v3.3.0)
- name: dplyr
  role: Used for data manipulation when pooling significant ASV calls per tool and
    tallying counts (v0.8.5)
- name: run_all_tools.sh
  role: Produces the per-dataset, per-tool significant ASV result tables that are
    aggregated (docker image dockerdkd/hackathon2021)
  repo: https://hub.docker.com/r/dockerdkd/hackathon2021
- name: ALDEx2
  role: One of the 14 DA tools whose significant-ASV calls are pooled; expected among
    the most consistent tools
- name: ANCOM-II
  role: One of the 14 DA tools whose significant-ASV calls are pooled; expected among
    the most consistent tools and to agree best with the intersect of tool results
  repo: FrederickHuangLin/ANCOM-Code-Archive
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:218bc6846c32ad1ae495b1ca1c9230564400c833f299ff33bc73ea58ccce9852
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ALDEx2
  - ANCOM-II
  - dplyr
  - ggplot2
  - parallelDist
  - run_all_tools.sh
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# microbiome-differential-abundance-results-aggregation

## Summary

Aggregates significant-ASV calls from multiple differential abundance (DA) tools across 16S rRNA datasets to quantify cross-tool overlap and clustering, using tool-vs-tool Jaccard distances and PCoA ordination, and to measure how prevalence filtering (e.g., -F 0.1) changes agreement.

## When to use

Use when you have significant-ASV result tables from several DA tools (e.g., 14 tools) run on the same datasets and need to quantify overlap, consensus, or tool clustering, including effects of prevalence filtering.

## When NOT to use

- You are estimating false-positive rates on randomized/simulated data rather than aggregating real significant-ASV calls (the paper handles this with separate run_all_FP_datasets_filt.sh / run_all_FP_datasets_unfilt.sh scripts)
- You are assessing cross-study consistency of genera across datasets from the same environment (e.g., the diarrhea or obesity consistency analyses), which uses a different random-expectation comparison rather than tool-vs-tool Jaccard distances
- You only ran a single DA tool per dataset, so there is no cross-tool overlap to aggregate

## Inputs

- Per-dataset, per-tool significant ASV result tables from run_all_tools.sh on the 38 filtered datasets (run_0.1_filt_datasets.sh)
- Per-dataset, per-tool significant ASV result tables from run_all_tools.sh on the 38 unfiltered datasets (run_no_filt_datasets.sh)
- ASV count table (ASV_TABLE.tsv) and two-column metadata file (METADATA_FILE.tsv) with sample names and groupings (max 2 groupings), used to generate the tool outputs

## Outputs

- Tool-tally table (consensus_asv_tool_counts.tsv) counting how many of the 14 tools called each ASV significant, per filtering condition
- PCoA ordination figure (fig3_pcoa_tool_jaccard.png) of tools based on mean pairwise Jaccard distances of their significant-ASV sets
- Mean Jaccard distance matrix averaged across the 38 datasets

## How to apply

Load the per-dataset, per-tool significant ASV tables produced by run_all_tools.sh on the 38 filtered datasets (run_0.1_filt_datasets.sh, which uses a 10% prevalence filter, i.e. -F 0.1 with rarefaction depth -D 2000) and the 38 unfiltered datasets (run_no_filt_datasets.sh). For each dataset, pool the significant ASV calls per tool and tally, for each ASV, how many of the 14 tools called it significantly DA, separately for filtered and unfiltered runs. Compute pairwise Jaccard distances between the significant-ASV sets of every tool pair with the R parallelDist package (v0.2.4) and average the distances across the 38 datasets. Run principal coordinates analysis on the mean Jaccard distance matrix with vegan (v2.5.6) in R 3.6.3, then plot the ordination with ggplot2 (v3.3.0) labeling tools by clustering behavior (e.g., edgeR/LEfSe grouping). Interpret ALDEx2 and ANCOM-II as expected to be the most consistent and to agree best with the intersect of tool results.

## Related tools

- **parallelDist** (Computes pairwise Jaccard distances between the significant-ASV sets of every tool pair (v0.2.4))
- **vegan** (Runs principal coordinates analysis on the mean tool-by-tool Jaccard distance matrix (v2.5.6, R 3.6.3))
- **ggplot2** (Plots the PCoA ordination labeling tools by clustering behavior (v3.3.0))
- **dplyr** (Used for data manipulation when pooling significant ASV calls per tool and tallying counts (v0.8.5))
- **run_all_tools.sh** (Produces the per-dataset, per-tool significant ASV result tables that are aggregated (docker image dockerdkd/hackathon2021)) — https://hub.docker.com/r/dockerdkd/hackathon2021
- **ALDEx2** (One of the 14 DA tools whose significant-ASV calls are pooled; expected among the most consistent tools)
- **ANCOM-II** (One of the 14 DA tools whose significant-ASV calls are pooled; expected among the most consistent tools and to agree best with the intersect of tool results) — FrederickHuangLin/ANCOM-Code-Archive

## Examples

```
dist <- parallelDist::parallelDist(t(sig_asv_matrix), method='binary'); ord <- vegan::cmdscale(dist, k=2)
```

## Evaluation signals

- In prevalence-filtered data, the proportion of significant ASVs identified by more than 12 tools should be much higher (mean 38.6%, SD 15.8%) than in unfiltered data (mean 17.3%, SD 22.1%)
- The distribution of the per-ASV tool-call counts should show most ASVs being called by either many tools or few tools in the filtered runs
- The mean Jaccard distance matrix must be computed as the average of the 38 per-dataset distance matrices, not from pooled raw calls
- ALDEx2 and ANCOM-II should cluster together and be among the most consistent tools in the PCoA ordination; edgeR and LEfSe are expected to group together
- All 38 filtered and 38 unfiltered datasets must be present in the tally before averaging; missing tool outputs would skew the per-ASV counts

## Limitations

- The consensus intersection of tool outputs is not guaranteed to be more reliable; the tools may simply be picking up the same noise as significant
- The 10% prevalence filter cut-off was arbitrarily selected; more work is needed to estimate an optimal cut-off
- Not all environments and datasets are appropriate for cross-data consistency comparisons; differences between datasets could reflect technical or biological factors
- It is unclear why some methods have much higher significance rates than others

## Evidence

- [abstract] ALDEx2 and ANCOM-II produce the most consistent results across studies and agree best with the intersect of results from different approaches.: "ALDEx2 and ANCOM-II produce the most consistent results across studies and agree best with the intersect of results from different approaches."
- [methods] Filtering run command: "run_all_tools.sh -A ASV_TABLE.tsv -G METADATA_FILE.tsv -O OUTPUT_FOLDER -D 2000 -F 0.1"
- [discussion] Recommend consensus approach: "we recommend that users employ several methods and focus on significant features identified by most tools, while keeping in mind the characteristics of the tools presented within this manuscript."
- [discussion] Caveat on intersections: "there is no assurance that the intersection of the tool outputs is more reliable; it is possible that the tools are simply picking up the same noise as significant."
- [readme] Filtered datasets script: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [readme] Prevalence filter definition: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
