---
name: subnetwork-hub-taxa-identification
description: Use when a taxa-by-taxon co-occurrence network has been partitioned into
  subnetworks (communities) and you need to rank taxa by connectivity to flag central
  hub taxa per subnetwork.
when_to_use_negative:
- You need direct ecological interaction evidence — the article states co-occurrence
  may reflect 'a common response to an external driver' rather than predator–prey
  relationships
- Input is raw amplicon sequencing data or a pre-network feature table; network construction
  and subnetwork clustering must be completed first
- Your dataset has no community/subnetwork structure to rank within (a single global
  network without seasonal community partitioning)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3299
tools:
- name: R
  role: Language used to load subnetwork edge lists, compute node degrees/connectivity,
    and write connectivity_summary.csv
  repo: https://github.com/marinebon/eDNA_microbes_whales
- name: igraph
  role: R library used to build graph objects for the blue and grey subnetworks and
    compute per-taxon degree and edge-weight sums
  repo: https://github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:d888fef0a3803fd530160a301b56465eabf035b18724340e4ad450cd4a95c881
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  - igraph
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3299
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# subnetwork-hub-taxa-identification

## Summary

Identifies the single most highly connected (central) hub taxon within each seasonal subnetwork (e.g. autumn 'blue' and winter 'grey') derived from the 18-month Monterey Bay eDNA community network, quantifying each hub's degree and its percentage of subnetwork taxa.

## When to use

Use when a taxa-by-taxon co-occurrence network has been partitioned into subnetworks (communities) and you need to rank taxa by connectivity to flag central hub taxa per subnetwork.

## When NOT to use

- You need direct ecological interaction evidence — the article states co-occurrence may reflect 'a common response to an external driver' rather than predator–prey relationships
- Input is raw amplicon sequencing data or a pre-network feature table; network construction and subnetwork clustering must be completed first
- Your dataset has no community/subnetwork structure to rank within (a single global network without seasonal community partitioning)

## Inputs

- Edge lists for each subnetwork (blue/autumn and grey/winter) from the Monterey Bay eDNA community network analysis (github.com/marinebon/eDNA_microbes_whales)
- Node/taxon attribute table with taxonomic assignments (Family or higher rank) and trophic-level annotations (e.g. tertiary consumer flags)

## Outputs

- connectivity_summary.csv with per-taxon connectivity ranks, node degree, edge-weight sums, percentage of subnetwork taxa connected, and tertiary-consumer flags
- Hub taxon per subnetwork with degree (blue: Balaenopteridae 24/29%; grey: Rhodospirillaceae 48/42%)

## How to apply

Load the blue (autumn) and grey (winter) subnetwork edge lists from the eDNA_microbes_whales repository and build igraph graph objects for each. For every taxon, compute connectivity as node degree (and optionally summed edge weights) within its subnetwork, then rank taxa and flag the top 10% most-connected taxa, marking tertiary consumers such as Balaenopteridae among them. Identify the top-ranked taxon per subnetwork as the hub and record its degree and the percentage of subnetwork taxa it connects to. Validate against the paper's reported values: Balaenopteridae connected to 24 taxa (29%) in the blue subnetwork and Rhodospirillaceae to 48 taxa (42%) in the grey subnetwork. Write the connectivity ranks, degree/edge-weight sums, percentage-of-subnetwork connected, and tertiary-consumer flags to connectivity_summary.csv. Remember the paper's caveat that network connectivity does not necessarily indicate direct interaction — a more connected taxon is more likely to show similar abundance patterns.

## Related tools

- **R** (Language used to load subnetwork edge lists, compute node degrees/connectivity, and write connectivity_summary.csv) — https://github.com/marinebon/eDNA_microbes_whales
- **igraph** (R library used to build graph objects for the blue and grey subnetworks and compute per-taxon degree and edge-weight sums) — https://github.com/marinebon/eDNA_microbes_whales

## Examples

```
Rscript -e 'library(igraph); g.blue <- graph_from_data_frame(read.csv("blue_edges.csv")); hub <- V(g.blue)[degree(g.blue)==max(degree(g.blue))]'
```

## Evaluation signals

- Computed degree of Balaenopteridae in the blue subnetwork equals 24 connections, i.e. 29% of blue subnetwork taxa
- Computed degree of Rhodospirillaceae in the grey subnetwork equals 48 connections, i.e. 42% of grey subnetwork taxa
- Exactly one hub taxon flagged per subnetwork, ranked highest by node degree within its community
- Top-10% connectivity flags computed per subnetwork (not across the full network) and tertiary consumers among them identified
- connectivity_summary.csv contains rank, degree, edge-weight sum, percentage-connected, and tertiary-consumer flag columns for every taxon in each subnetwork

## Limitations

- Whether taxon co-occurrence is driven by direct predator–prey relationships or a common response to an external driver is unclear
- Network connectivity does not necessarily indicate interaction; a more connected taxon is only more likely to present similar patterns of abundance
- Primer biases, inhibitors, and variable template abundances affect which taxa are detected, constraining the taxon set available for network ranking

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Network connectivity caveat: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [discussion] Co-occurrence interpretation caveat: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [readme] README scope statement: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
