---
name: dynamic-tree-cut-module-detection
description: Use when a taxon-by-sample abundance time-series (e.g. 663 taxonomic
  groups across an 18-month Monterey Bay 2015–2016 eDNA index table) should be partitioned
  into co-fluctuating subnetworks/modules with seasonal membership.
when_to_use_negative:
- The abundance table has no temporal replication (single samples), so co-fluctuation
  modules and seasonal membership cannot be defined
- The goal is to claim direct predator–prey interactions from module connectivity
  — the article cautions that network connectivity does not necessarily indicate interaction
  and co-occurrence may reflect a common response to an external driver
- Species-level module interpretation is required — the article notes the lower the
  taxonomic assignment, the lower the confidence
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: WGCNA (R package)
  role: Builds the weighted taxon correlation network (beta=22), computes topological
    overlap, and supplies the clustering used for Dynamic Tree Cut module detection
  repo: github.com/marinebon/eDNA_microbes_whales
- name: R
  role: Language used to load the eDNA index table from the repository and run the
    WGCNA module-detection workflow
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:f5a5fecb48bacaec8b18300ae0d17f4cf50a0fedb5ea7b8c64f92baba6d29c02
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  - WGCNA (R package)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dynamic-tree-cut-module-detection

## Summary

Detects taxon modules (subnetworks) in eDNA amplicon-index time-series by building a weighted correlation network (WGCNA, soft-thresholding power beta=22), converting correlations to topological-overlap dissimilarity, and cutting the hierarchical clustering dendrogram with the Dynamic Tree Cut algorithm. It is what allowed Djurhuus et al. to identify six seasonal taxon communities from 663 Monterey Bay taxonomic groups.

## When to use

Use when a taxon-by-sample abundance time-series (e.g. 663 taxonomic groups across an 18-month Monterey Bay 2015–2016 eDNA index table) should be partitioned into co-fluctuating subnetworks/modules with seasonal membership.

## When NOT to use

- The abundance table has no temporal replication (single samples), so co-fluctuation modules and seasonal membership cannot be defined
- The goal is to claim direct predator–prey interactions from module connectivity — the article cautions that network connectivity does not necessarily indicate interaction and co-occurrence may reflect a common response to an external driver
- Species-level module interpretation is required — the article notes the lower the taxonomic assignment, the lower the confidence

## Inputs

- eDNA amplicon abundance index table: 663 taxonomic groups (Family or higher rank) across an 18-month (2015–2016) Monterey Bay seawater time-series
- Soft-thresholding power beta=22 for WGCNA network construction

## Outputs

- subnetwork_members.csv: table mapping each taxon to its color-labeled module (grey, blue, black, green, yellow, orange) and seasonal membership
- Taxon-by-taxon topological overlap dissimilarity matrix and clustering dendrogram

## How to apply

Load the eDNA amplicon abundance index table from the marinebon/eDNA_microbes_whales repository with pandas/R. Apply WGCNA to the abundance indices with soft-thresholding power beta=22 to obtain the taxon-by-taxon correlation structure, then convert correlations to a topological-overlap dissimilarity matrix and cluster hierarchically. Cut the dendrogram with the Dynamic Tree Cut (cutreeDynamic) algorithm to detect the six subnetworks (modules). Assign each module a color label (grey, blue, black, green, yellow, orange) and a season based on the temporal abundance pattern of its member taxa. Write the module membership table to subnetwork_members.csv and compare against the Fig. 1 assignments.

## Related tools

- **WGCNA (R package)** (Builds the weighted taxon correlation network (beta=22), computes topological overlap, and supplies the clustering used for Dynamic Tree Cut module detection) — github.com/marinebon/eDNA_microbes_whales
- **R** (Language used to load the eDNA index table from the repository and run the WGCNA module-detection workflow) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
library(WGCNA); net <- blockwiseModules(t(otus), power = 22, deepSplit = 2, pamRespectsDendro = FALSE); write.csv(net$colors, "subnetwork_members.csv")
```

## Evaluation signals

- Exactly six modules (subnetworks) are detected from the 663-taxa time-series, matching the color labels reported in Fig. 1
- Color–season assignments reproduce: blue=autumn with n=81 taxa, grey=winter with n=114 taxa, and orange/yellow/green assigned to spring/summer
- eDNA richness observations separate the spring/summer subnetworks (orange, yellow, green) from the autumn/winter subnetworks (grey, blue, black), confirming coherent seasonal membership
- Module assignment every taxon to exactly one color module, exportable as a valid subnetwork_members.csv membership table
- The correlation network uses the documented soft-thresholding power beta=22 before topological-overlap computation

## Limitations

- Network connectivity does not necessarily indicate interaction; a more connected taxon is only more likely to present similar abundance patterns
- Whether taxon co-occurrence reflects direct predator–prey relationships or a common response to an external driver is unclear
- Amplification is subject to primer biases, inhibitors, and variable template abundances, affecting the number of taxa detected and thus module composition
- Hypotheses from the network analyses require further hypothesis-based research and experiments; longer time-series are needed to increase power and validity
- No method, including this one, identifies all taxa in an ecosystem

## Evidence

- [other] six seasonal subnetworks: [evidence span withheld — anchor status: not_found]
- [discussion] clustering into subnetworks: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [abstract] input dataset scale: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [readme] repository role: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
