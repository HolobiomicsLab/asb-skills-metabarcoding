---
name: network-metrics-verification-against-reported-values
description: Use when reproducing or validating reported hub-taxa connectivity (degrees
  and percent-of-subnetwork) from seasonal subnetwork edge lists derived from the
  Monterey Bay 18-month eDNA time-series.
when_to_use_negative:
- Your goal is species-level resolution — the underlying dataset assigns taxa at Family
  or higher rank, where confidence in assignment decreases with finer taxonomy
- You need demonstrated biological interactions — co-occurrence in the network may
  reflect a common response to an external driver rather than direct predator-prey
  relationships
- Your input is raw sequencing data rather than a derived subnetwork edge list — this
  skill operates on network outputs, not FASTQ or amplicon reads
tools:
- name: R igraph
  role: Build graph objects from the blue/grey subnetwork edge lists and compute node
    degrees and edge-weight sums for per-taxon connectivity ranking
  repo: github.com/marinebon/eDNA_microbes_whales
- name: eDNA_microbes_whales
  role: Source repository providing the subnetwork edge lists and analysis scripts
    derived from the Monterey Bay eDNA time-series community network analysis
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:878f22579fb0a6d33362bb6c4bb973faf30dad73f4f25fa0610824b95a9884aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  - R igraph
  - eDNA_microbes_whales
  - igraph
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# network-metrics-verification-against-reported-values

## Summary

Recompute per-taxon connectivity (node degree and edge-weight sums) in the seasonal subnetworks from an eDNA community network analysis and verify that computed hub values match the reported ones (Balaenopteridae 24 taxa/29% in the blue subnetwork; Rhodospirillaceae 48 taxa/42% in the grey subnetwork) before downstream reuse.

## When to use

Use when reproducing or validating reported hub-taxa connectivity (degrees and percent-of-subnetwork) from seasonal subnetwork edge lists derived from the Monterey Bay 18-month eDNA time-series.

## When NOT to use

- Your goal is species-level resolution — the underlying dataset assigns taxa at Family or higher rank, where confidence in assignment decreases with finer taxonomy
- You need demonstrated biological interactions — co-occurrence in the network may reflect a common response to an external driver rather than direct predator-prey relationships
- Your input is raw sequencing data rather than a derived subnetwork edge list — this skill operates on network outputs, not FASTQ or amplicon reads

## Inputs

- Blue (autumn) subnetwork edge list from the eDNA_microbes_whales repository
- Grey (winter) subnetwork edge list from the eDNA_microbes_whales repository
- Taxon-to-trophic-role annotations (for tertiary-consumer flags)

## Outputs

- connectivity_summary.csv with per-taxon connectivity rank, degree, edge-weight sum, percent-of-subnetwork connected, and tertiary-consumer flag
- Pass/fail validation report comparing computed degrees to reported values (blue: 24/29%; grey: 48/42%)

## How to apply

Load the blue (autumn) and grey (winter) subnetwork edge lists from github.com/marinebon/eDNA_microbes_whales, which derives from the 18-month Monterey Bay eDNA time-series community network analysis. Build igraph graph objects for each subnetwork and compute per-taxon connectivity as node degree plus the sum of edge weights within each subnetwork. Rank taxa by connectivity, flag the top 10% most-connected taxa, and mark tertiary consumers among them. Treat a computation as validated only when Balaenopteridae's degree in the blue subnetwork equals 24 (29% of subnetwork taxa) and Rhodospirillaceae's degree in the grey subnetwork equals 48 (42%), matching the reported findings. Write connectivity ranks, degree/edge-weight sums, percentage-of-subnetwork connected, and tertiary-consumer flags to connectivity_summary.csv.

## Related tools

- **R igraph** (Build graph objects from the blue/grey subnetwork edge lists and compute node degrees and edge-weight sums for per-taxon connectivity ranking) — github.com/marinebon/eDNA_microbes_whales
- **eDNA_microbes_whales** (Source repository providing the subnetwork edge lists and analysis scripts derived from the Monterey Bay eDNA time-series community network analysis) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
Rscript -e 'library(igraph); g <- graph_from_data_frame(read.csv("blue_edges.csv")); degree(g)["Balaenopteridae"]'
```

## Evaluation signals

- Computed degree of Balaenopteridae in the blue subnetwork equals 24 connections (29% of subnetwork taxa)
- Computed degree of Rhodospirillaceae in the grey subnetwork equals 48 connections (42% of subnetwork taxa)
- connectivity_summary.csv contains rank, degree, edge-weight sum, percent connected, and tertiary-consumer flag columns populated for all taxa
- Top-10% connectivity threshold flags exactly the reported hub taxa (Balaenopteridae in blue, Rhodospirillaceae in grey) as rank-1 in their respective subnetworks
- Percentage-of-subnetwork values are consistent with the taxon count of each subnetwork (24/0.29 and 48/0.42 imply the blue and grey subnetwork sizes)

## Limitations

- Network connectivity does not necessarily indicate interaction; a more connected taxon is only more likely to present similar patterns of abundance
- Whether co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- No method, including this one, identifies all taxa in an ecosystem, and amplification is subject to primer biases, inhibitors, and variable template abundances
- The repository README is minimal and does not document edge-list formats or file locations, so inputs must be located within the repository

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time.: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [discussion] Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
