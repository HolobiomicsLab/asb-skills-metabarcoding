---
name: seasonal-subnetwork-annotation
description: Use when you have a taxon-by-time abundance index matrix (e.g., 663 taxonomic
  groups across an 18-month seawater eDNA series) and need to detect correlated-abundance
  community modules and label them by season.
when_to_use_negative:
- The input is a single-sample amplicon dataset with no temporal replication — WGCNA
  module detection over correlated abundance shifts requires a time-series of many
  samples
- The goal is species-level taxonomic assignment or biodiversity counting rather than
  community-level network structure — the article notes species-level assignment has
  lower confidence and that this skill addresses community clustering, not taxonomy
- The abundance table is already clustered or annotated (e.g., an existing module
  membership file) — rerunning WGCNA would discard prior assignments
edam_operation: http://edamontology.org/operation_3432
tools:
- name: WGCNA
  role: R package used to build the taxon correlation network (soft-thresholding power
    beta=22), compute the topological overlap dissimilarity matrix, and support Dynamic
    Tree Cut module detection
- name: R
  role: Environment for loading the abundance index table and running the WGCNA pipeline
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:e153862a7a75890e5213de435e7008feaad4f3aa59bf60c662ddc886cd0678d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# seasonal-subnetwork-annotation

## Summary

Clusters taxa from an eDNA amplicon time-series into seasonal community subnetworks using WGCNA (soft-thresholding power beta=22, topological-overlap hierarchical clustering, Dynamic Tree Cut), then assigns each module a color label and seasonal membership as reported in Djurhuus et al. 2020.

## When to use

Use when you have a taxon-by-time abundance index matrix (e.g., 663 taxonomic groups across an 18-month seawater eDNA series) and need to detect correlated-abundance community modules and label them by season.

## When NOT to use

- The input is a single-sample amplicon dataset with no temporal replication — WGCNA module detection over correlated abundance shifts requires a time-series of many samples
- The goal is species-level taxonomic assignment or biodiversity counting rather than community-level network structure — the article notes species-level assignment has lower confidence and that this skill addresses community clustering, not taxonomy
- The abundance table is already clustered or annotated (e.g., an existing module membership file) — rerunning WGCNA would discard prior assignments

## Inputs

- eDNA amplicon abundance index table: 663 taxonomic groups (Family or higher rank) sampled across the 18-month Monterey Bay 2015–2016 seawater time-series, loadable with pandas or R from the marinebon/eDNA_microbes_whales repository

## Outputs

- subnetwork_members.csv: module membership table mapping each of the 663 taxa to one of six color-labeled subnetworks (grey, blue, black, green, yellow, orange) with seasonal assignment
- six color–season module labels derived from the temporal abundance patterns of member taxa

## How to apply

Load the eDNA amplicon abundance index table (663 taxonomic groups, Monterey Bay 2015–2016 18-month time series) from the marinebon/eDNA_microbes_whales repository. Build the taxon-by-taxon correlation structure with WGCNA using soft-thresholding power beta=22 on the abundance indices. Convert correlations to a topological overlap dissimilarity matrix, perform hierarchical clustering, and cut the dendrogram with the Dynamic Tree Cut algorithm to recover the six modules. Assign each module a color label and a seasonal membership (e.g., blue=autumn with n=81 taxa, grey=winter with n=114 taxa) based on the temporal abundance pattern of its member taxa. The decision point is verifying that exactly six subnetworks emerge and that spring/summer modules (orange, yellow, green) separate from autumn/winter modules (grey, blue, black) before writing subnetwork_members.csv.

## Related tools

- **WGCNA** (R package used to build the taxon correlation network (soft-thresholding power beta=22), compute the topological overlap dissimilarity matrix, and support Dynamic Tree Cut module detection)
- **R** (Environment for loading the abundance index table and running the WGCNA pipeline) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
Rscript -e 'library(WGCNA); net <- blockwiseModules(datExpr, power = 22); write.csv(net$colors, "subnetwork_members.csv")'
```

## Evaluation signals

- Exactly six subnetworks (modules) are detected after dendrogram cutting with Dynamic Tree Cut
- Color–season assignments match Fig. 1 of the paper (e.g., blue=autumn, grey=winter)
- Module sizes match reported values, e.g., blue module n=81 taxa, grey module n=114 taxa
- eDNA richness observations separate spring/summer subnetworks (orange, yellow, green) from autumn/winter subnetworks (grey, blue, black), confirming coherent seasonal membership
- subnetwork_members.csv is produced and every one of the 663 taxonomic groups is assigned a module label

## Limitations

- Whether taxon co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- Network connectivity does not necessarily indicate interaction
- Amplification is subject to primer biases, inhibitors, and variable template abundances, which affect taxa detection; no method identifies all taxa in an ecosystem
- Predicted taxon interdependencies require further hypothesis-based research; longer time-series are needed to increase power and validity

## Evidence

- [results] eDNA richness separates seasonal subnetworks: [evidence span withheld — anchor status: not_found]
- [discussion] Marine organisms cluster into communities over time: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [abstract] Dataset scope: 663 taxonomic groups: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [discussion] Network connectivity caveat: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [readme] Repository purpose (README): "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
