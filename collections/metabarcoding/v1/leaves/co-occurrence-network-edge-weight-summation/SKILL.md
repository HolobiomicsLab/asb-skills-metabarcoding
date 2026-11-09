---
name: co-occurrence-network-edge-weight-summation
description: Use when you have per-subnetwork edge lists (e.g. blue/grey seasonal
  subnetworks from a co-occurrence network) and need to rank taxa by connectivity
  to flag central hub taxa and their degree as a percentage of subnetwork taxa.
when_to_use_negative:
- Input is raw amplicon sequencing data (FASTQ) or an unclustered OTU/asv table —
  the skill operates on already-derived subnetwork edge lists, not on sequencing output
- The goal is to confirm direct predator–prey interactions — the paper notes co-occurrence
  may be 'a common response to an external driver' and requires hypothesis-based follow-up
  experiments
- Species-level hub identification is required — the paper warns that 'the lower the
  taxonomy assignment, the lower the confidence in that assignment becomes' (findings
  are at Family or higher rank)
edam_operation: http://edamontology.org/operation_3925
tools:
- name: igraph
  role: Builds graph objects for each subnetwork and computes per-taxon node degree
    and edge-weight sums used for connectivity ranking
- name: R
  role: Scripting environment in which the igraph-based connectivity ranking and CSV
    export are implemented
- name: eDNA_microbes_whales
  role: Source repository supplying the blue and grey subnetwork edge lists from the
    Monterey Bay eDNA time-series network analysis
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:979fce55979756d0bfa26ccfe8ee3ecbbdd8f798140f590dd6aa665987fe5370
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - eDNA_microbes_whales
  - igraph
  edam_operation: http://edamontology.org/operation_3925
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# co-occurrence-network-edge-weight-summation

## Summary

Ranks taxa in the blue (autumn) and grey (winter) subnetworks of the Monterey Bay 18-month eDNA community network by summed edge weights and node degree to identify the single most-connected hub taxon in each subnetwork. It reproduces the paper's reported hubs (Balaenopteridae, Rhodospirillaceae) and their connection counts as validation targets.

## When to use

Use when you have per-subnetwork edge lists (e.g. blue/grey seasonal subnetworks from a co-occurrence network) and need to rank taxa by connectivity to flag central hub taxa and their degree as a percentage of subnetwork taxa.

## When NOT to use

- Input is raw amplicon sequencing data (FASTQ) or an unclustered OTU/asv table — the skill operates on already-derived subnetwork edge lists, not on sequencing output
- The goal is to confirm direct predator–prey interactions — the paper notes co-occurrence may be 'a common response to an external driver' and requires hypothesis-based follow-up experiments
- Species-level hub identification is required — the paper warns that 'the lower the taxonomy assignment, the lower the confidence in that assignment becomes' (findings are at Family or higher rank)

## Inputs

- Blue (autumn) subnetwork edge list from the Monterey Bay eDNA community network analysis
- Grey (winter) subnetwork edge list from the same network analysis
- Taxon labels at Family-or-higher rank from the 663-taxonomic-group eDNA dataset
- Tertiary-consumer annotations for flagging top-ranked taxa

## Outputs

- connectivity_summary.csv with per-taxon connectivity rank, degree, edge-weight sum, percentage-of-subnetwork connected, and tertiary-consumer flags
- Identified hub taxa per subnetwork (expected: Balaenopteridae, 24 connections/29% in blue; Rhodospirillaceae, 48 connections/42% in grey)

## How to apply

Load the blue and grey subnetwork edge lists from the eDNA_microbes_whales repository (github.com/marinebon/eDNA_microbes_whales), which derives from the multilocus amplicon sequencing of an eighteen-month (2015–2016) Monterey Bay seawater time-series. Build graph objects for each subnetwork with igraph in R and compute per-taxon connectivity as the sum of edge weights plus node degree within each subnetwork. Rank taxa by connectivity and flag the top 10% most-connected taxa in each subnetwork, marking tertiary consumers among them. Verify the computed hubs against the reported values: Balaenopteridae connected to 24 taxa (29%) in the blue subnetwork and Rhodospirillaceae connected to 48 taxa (42%) in the grey subnetwork. Write connectivity ranks, degree/edge-weight sums, percentage-of-subnetwork connected, and tertiary-consumer flags to connectivity_summary.csv.

## Related tools

- **igraph** (Builds graph objects for each subnetwork and computes per-taxon node degree and edge-weight sums used for connectivity ranking)
- **R** (Scripting environment in which the igraph-based connectivity ranking and CSV export are implemented)
- **eDNA_microbes_whales** (Source repository supplying the blue and grey subnetwork edge lists from the Monterey Bay eDNA time-series network analysis) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
library(igraph); g <- graph_from_data_frame(blue_edges); hub <- names(sort(degree(g), decreasing=TRUE))[1]; stopifnot(degree(g, hub) == 24)
```

## Evaluation signals

- Computed degree of Balaenopteridae in the blue subnetwork equals exactly 24 connections, which is 29% of the blue subnetwork's taxa
- Computed degree of Rhodospirillaceae in the grey subnetwork equals exactly 48 connections, which is 42% of the grey subnetwork's taxa
- Balaenopteridae and Rhodospirillaceae are the top-ranked (most-connected) taxa in their respective subnetworks
- connectivity_summary.csv contains all four required columns: connectivity rank, degree/edge-weight sum, percentage-of-subnetwork connected, and tertiary-consumer flag
- The flagged top 10% most-connected taxa in each subnetwork include the marked tertiary consumers

## Limitations

- Network connectivity does not necessarily indicate interaction; a more connected taxon is only more likely to present similar patterns of abundance
- Whether co-occurrence reflects direct predator–prey relationships or a common response to an external driver is unclear; results need hypothesis-based follow-up experiments
- Amplification is subject to primer biases, inhibitors, and variable template abundances, affecting which taxa are detected; no method identifies all taxa in an ecosystem
- eDNA cannot identify life-history stages and may include sequences from dead or decaying biological matter
- The repository README states it is intended for the eDNA group's own analysis, quality filtering, and bioinformatics changes, so edge-list provenance should be checked before reuse

## Evidence

- [results] Balaenopteridae in blue / Rhodospirillaceae hub finding: [evidence span withheld — anchor status: not_found]
- [abstract] Network analysis scope: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Connectivity caveat: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [readme] README purpose statement: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
