---
name: trophic-level-taxa-classification
description: Use when a taxon-by-taxon network (edge lists from eDNA co-occurrence/subnetworks)
  needs hub-taxon identification and trophic-level flags, e.g. tertiary consumers
  among the top 10% most-connected taxa per subnetwork.
when_to_use_negative:
- Input is a raw sequence file or amplicon reads rather than a derived taxon network/edge
  list — upstream bioinformatics and network construction must come first
- 'The analysis goal is validating direct predator-prey interactions: the paper notes
  taxon co-occurrence may reflect a common response to an external driver, and network
  connectivity does not necessarily indicate interaction'
- 'Species-level trophic assignment is required: the article notes the lower the taxonomic
  assignment (Family or higher here), the lower the confidence'
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: R
  role: Language used to build subnetwork graphs, compute per-taxon connectivity (degree,
    edge-weight sums), rank taxa, and write connectivity_summary.csv
- name: igraph
  role: Graph library used to construct graph objects for the blue and grey subnetworks
    and compute node degree / connectivity ranks
- name: eDNA_microbes_whales
  role: Source repository supplying the blue and grey subnetwork edge lists from the
    Monterey Bay eDNA community network analysis
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:b21a75e5bf1712e47d6e4496b88d0302fd85472f38bfdc6f5d8fae94973fa213
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  - eDNA_microbes_whales
  - igraph
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# trophic-level-taxa-classification

## Summary

Classifies taxa in an eDNA community network by trophic role (e.g., flagging tertiary consumers) and identifies the single most highly connected hub taxon per seasonal subnetwork, ranking connectivity as node degree and percent of subnetwork taxa connected.

## When to use

Use when a taxon-by-taxon network (edge lists from eDNA co-occurrence/subnetworks) needs hub-taxon identification and trophic-level flags, e.g. tertiary consumers among the top 10% most-connected taxa per subnetwork.

## When NOT to use

- Input is a raw sequence file or amplicon reads rather than a derived taxon network/edge list — upstream bioinformatics and network construction must come first
- The analysis goal is validating direct predator-prey interactions: the paper notes taxon co-occurrence may reflect a common response to an external driver, and network connectivity does not necessarily indicate interaction
- Species-level trophic assignment is required: the article notes the lower the taxonomic assignment (Family or higher here), the lower the confidence

## Inputs

- Blue (autumn) subnetwork edge list (taxon-taxon connections) from the eDNA_microbes_whales repository
- Grey (winter) subnetwork edge list (taxon-taxon connections) from the eDNA_microbes_whales repository
- Taxon metadata for trophic-level assignment (e.g., tertiary-consumer status)

## Outputs

- connectivity_summary.csv with per-taxon connectivity ranks, degree/edge-weight sums, percentage of subnetwork connected, and tertiary-consumer flags
- Identified hub taxa per subnetwork (Balaenopteridae: 24 taxa/29% in blue; Rhodospirillaceae: 48 taxa/42% in grey)

## How to apply

Load the blue and grey subnetwork edge lists from the eDNA_microbes_whales repository (github.com/marinebon/eDNA_microbes_whales), derived from the 18-month Monterey Bay eDNA time-series community network analysis. Build graph objects for each subnetwork with igraph in R and compute per-taxon connectivity as node degree (sum of connections) within each subnetwork. Rank taxa by connectivity, flag the top 10% most-connected taxa in each subnetwork, and mark tertiary consumers among them. Verify hubs against reported values: Balaenopteridae connected to 24 taxa (29%) in the blue subnetwork and Rhodospirillaceae connected to 48 taxa (42%) in the grey subnetwork. Write ranks, degree/edge-weight sums, percent-of-subnetwork connected, and tertiary-consumer flags to connectivity_summary.csv.

## Related tools

- **R** (Language used to build subnetwork graphs, compute per-taxon connectivity (degree, edge-weight sums), rank taxa, and write connectivity_summary.csv)
- **igraph** (Graph library used to construct graph objects for the blue and grey subnetworks and compute node degree / connectivity ranks)
- **eDNA_microbes_whales** (Source repository supplying the blue and grey subnetwork edge lists from the Monterey Bay eDNA community network analysis) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
library(igraph); g <- graph_from_data_frame(read.csv('blue_edges.csv')); d <- sort(degree(g), decreasing=TRUE); head(d)
```

## Evaluation signals

- Computed degree of Balaenopteridae in the blue subnetwork equals 24 connections, i.e. 29% of subnetwork taxa
- Computed degree of Rhodospirillaceae in the grey subnetwork equals 48 connections, i.e. 42% of subnetwork taxa
- Exactly the top 10% most-connected taxa are flagged in each subnetwork, with tertiary-consumer flags present among them
- connectivity_summary.csv contains non-null rank, degree/edge-weight sum, percent-connected, and tertiary-consumer columns for every taxon in both subnetworks
- Percent-connected values fall in the valid 0–100% range and hub percentages reproduce reported values on recomputation

## Limitations

- Network connectivity does not necessarily indicate interaction; a more connected taxon is only more likely to present similar patterns of abundance
- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- Taxonomic assignments are at Family or higher rank, so species-level trophic classification carries lower confidence
- No method, including this one, identifies all taxa in an ecosystem, affecting completeness of connectivity counts

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Network connectivity caveat: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [discussion] Co-occurrence driver caveat: "whether their co-occurrence in our study is driven by direct predator–prey relationships or is instead a common response to an external driver is unclear."
- [abstract] Dataset scope: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [readme] Repository purpose: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
