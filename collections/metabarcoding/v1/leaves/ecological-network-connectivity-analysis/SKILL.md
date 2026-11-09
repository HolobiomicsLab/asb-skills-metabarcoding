---
name: ecological-network-connectivity-analysis
description: Use when you have subnetwork edge lists from a co-occurrence/community
  network (e.g., seasonal subnetworks from an eDNA time series) and need to identify
  and rank the most highly connected hub taxa with degree/percentage-of-subnetwork
  statistics.
when_to_use_negative:
- Input is a raw sequencing dataset (FASTQ/amplicon reads) rather than a derived subnetwork
  edge list — the repository README indicates it is for sequencing data analysis and
  quality filtering, which precedes network construction
- The goal is species-level taxonomic assignment or life-history inference — the paper
  notes species-level assignments carry lower confidence and eDNA cannot identify
  life history stages
- You need to demonstrate direct predator–prey interaction — the paper states that
  whether co-occurrence reflects direct predator–prey relationships or a common response
  to an external driver is unclear
edam_operation: http://edamontology.org/operation_0272
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation language for loading subnetwork edge lists, building graph
    objects, ranking taxa by connectivity, and writing connectivity_summary.csv
  repo: github.com/marinebon/eDNA_microbes_whales
- name: igraph
  role: Builds graph objects from the blue and grey subnetwork edge lists and computes
    per-taxon degree and edge-weight sums used for hub ranking
  repo: github.com/marinebon/eDNA_microbes_whales
- name: eDNA_microbes_whales
  role: Source repository for the blue/grey subnetwork edge lists derived from the
    18-month Monterey Bay eDNA time-series community network analysis
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:adbe4bbc773373330b2a45d22a7f77a26ddb7c0c3760241e802547209c319f4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - eDNA_microbes_whales
  - igraph
  edam_operation: http://edamontology.org/operation_0272
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# ecological-network-connectivity-analysis

## Summary

Ranks hub taxa by connectivity (edge-weight sums and node degree) within seasonally clustered subnetworks of an eDNA community network, identifying the most central taxon in each subnetwork. Used to quantify, e.g., Balaenopteridae as the hub of the blue (autumn) subnetwork and Rhodospirillaceae as the hub of the grey (winter) subnetwork.

## When to use

Use when you have subnetwork edge lists from a co-occurrence/community network (e.g., seasonal subnetworks from an eDNA time series) and need to identify and rank the most highly connected hub taxa with degree/percentage-of-subnetwork statistics.

## When NOT to use

- Input is a raw sequencing dataset (FASTQ/amplicon reads) rather than a derived subnetwork edge list — the repository README indicates it is for sequencing data analysis and quality filtering, which precedes network construction
- The goal is species-level taxonomic assignment or life-history inference — the paper notes species-level assignments carry lower confidence and eDNA cannot identify life history stages
- You need to demonstrate direct predator–prey interaction — the paper states that whether co-occurrence reflects direct predator–prey relationships or a common response to an external driver is unclear

## Inputs

- Subnetwork edge lists (e.g., blue/autumn and grey/winter subnetworks) from the Monterey Bay eDNA community network
- Taxon abundance/taxonomic tables from the 18-month eDNA metabarcoding time-series (2015–2016)
- Trophic-level annotations (e.g., tertiary-consumer labels) for taxa

## Outputs

- connectivity_summary.csv with per-taxon connectivity ranks, degree, edge-weight sums, percentage-of-subnetwork connected, and tertiary-consumer flags
- Ranked hub-taxa lists per subnetwork (e.g., Balaenopteridae: 24 connections/29%; Rhodospirillaceae: 48 connections/42%)

## How to apply

Load the blue and grey subnetwork edge lists from the eDNA_microbes_whales repository (github.com/marinebon/eDNA_microbes_whales), which derives from the 18-month Monterey Bay eDNA time-series community network analysis. Build igraph graph objects for each subnetwork and compute per-taxon connectivity as node degree plus the sum of edge weights. Rank taxa by connectivity, flag the top 10% most-connected taxa in each subnetwork, and mark tertiary consumers among them. Validate against reported values: Balaenopteridae must connect to 24 taxa (29% of the blue subnetwork) and Rhodospirillaceae to 48 taxa (42% of the grey subnetwork). Interpret connectivity cautiously — the paper notes that a more connected taxon is more likely to show similar abundance patterns, but connectivity does not necessarily indicate direct interaction.

## Related tools

- **R** (Implementation language for loading subnetwork edge lists, building graph objects, ranking taxa by connectivity, and writing connectivity_summary.csv) — github.com/marinebon/eDNA_microbes_whales
- **igraph** (Builds graph objects from the blue and grey subnetwork edge lists and computes per-taxon degree and edge-weight sums used for hub ranking) — github.com/marinebon/eDNA_microbes_whales
- **eDNA_microbes_whales** (Source repository for the blue/grey subnetwork edge lists derived from the 18-month Monterey Bay eDNA time-series community network analysis) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
library(igraph); g <- graph_from_data_frame(read.csv('blue_subnetwork_edges.csv'), directed=FALSE); hubs <- sort(degree(g), decreasing=TRUE); write.csv(data.frame(taxon=names(hubs), degree=as.integer(hubs)), 'connectivity_summary.csv')
```

## Evaluation signals

- Computed degree of Balaenopteridae in the blue subnetwork equals 24 connections (29% of subnetwork taxa)
- Computed degree of Rhodospirillaceae in the grey subnetwork equals 48 connections (42% of subnetwork taxa)
- connectivity_summary.csv contains connectivity ranks, degree/edge-weight sums, percentage-of-subnetwork connected, and tertiary-consumer flags for every taxon in each subnetwork
- Top 10% most-connected taxa are flagged in each subnetwork, and tertiary consumers among them are marked
- Degrees and percentages are consistent per subnetwork (degree / number of taxa in subnetwork equals the reported percentage)

## Limitations

- Network connectivity does not necessarily indicate interaction; a more connected taxon is only more likely to present similar patterns of abundance
- Whether hub-taxa co-occurrence is driven by direct predator–prey relationships or a common response to an external driver is unclear
- Taxon interdependencies inferred from network analyses should be further investigated through hypothesis-based research and experiments
- Amplification is subject to primer biases, inhibitors, and variable template abundances, which affect which taxa are detected in the underlying data

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life"
- [discussion] Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [discussion] Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time.: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
