---
name: topological-overlap-matrix-clustering
description: Use when a taxon-by-sample abundance index table spanning many time points
  needs to be partitioned into correlated-abundance modules (e.g. subnetworks), starting
  from a taxon-by-taxon correlation structure with WGCNA soft-thresholding.
when_to_use_negative:
- Input is raw sequencing reads or an unnormalized amplicon table — the method requires
  the processed proportional-abundance index table, not FASTQ or filter-stage data
- Data are a single-snapshot community composition (no temporal dimension) — seasonal
  membership of modules cannot be assigned from temporal abundance patterns without
  a time-series
- The goal is detecting direct predator-prey interactions — the article notes that
  whether co-occurrence reflects direct interactions or a common response to an external
  driver is unclear, so modules should not be read as interaction evidence
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: WGCNA (R package)
  role: Builds the taxon-by-taxon correlation network with soft-thresholding power
    beta=22, computes the topological overlap dissimilarity matrix, and supports Dynamic
    Tree Cut module detection on the eDNA abundance indices
  repo: github.com/marinebon/eDNA_microbes_whales
- name: R
  role: Execution environment for the WGCNA pipeline and for writing the module membership
    table subnetwork_members.csv
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:306165b5eb33844d294745d86d2e38304fbfceb53125b581cce6d0d8145e3c78
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - WGCNA (R package)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# topological-overlap-matrix-clustering

## Summary

Applies WGCNA (weighted gene correlation network analysis) to an eDNA amplicon-index abundance time-series to cluster 663 Monterey Bay taxa into co-occurring subnetworks (modules) via a topological overlap dissimilarity matrix and dynamic tree cut, reproducing the paper's six seasonally-coherent communities.

## When to use

Use when a taxon-by-sample abundance index table spanning many time points needs to be partitioned into correlated-abundance modules (e.g. subnetworks), starting from a taxon-by-taxon correlation structure with WGCNA soft-thresholding.

## When NOT to use

- Input is raw sequencing reads or an unnormalized amplicon table — the method requires the processed proportional-abundance index table, not FASTQ or filter-stage data
- Data are a single-snapshot community composition (no temporal dimension) — seasonal membership of modules cannot be assigned from temporal abundance patterns without a time-series
- The goal is detecting direct predator-prey interactions — the article notes that whether co-occurrence reflects direct interactions or a common response to an external driver is unclear, so modules should not be read as interaction evidence

## Inputs

- eDNA amplicon abundance index table (taxa x time points; 663 taxonomic groups at Family or higher rank) from an 18-month Monterey Bay 2015-2016 seawater time-series
- soft-thresholding power beta (22 in this study)
- desired module count / dendrogram cut parameters for Dynamic Tree Cut

## Outputs

- subnetwork_members.csv — module membership table mapping each taxon to a color-labeled subnetwork with seasonal assignment
- six subnetworks with color labels (grey, blue, black, green, yellow, orange) and seasonal membership
- topological overlap dissimilarity matrix and hierarchical clustering dendrogram (intermediate objects)

## How to apply

Load the eDNA amplicon abundance index table (663 taxonomic groups across the 18-month Monterey Bay 2015–2016 seawater time-series) from the marinebon/eDNA_microbes_whales repository into R. Build the taxon-by-taxon correlation structure with WGCNA using soft-thresholding power beta=22, then convert correlations to a topological overlap dissimilarity matrix and perform hierarchical clustering on it. Cut the dendrogram with the Dynamic Tree Cut algorithm to detect the modules; the reference result is six subnetworks assigned color labels (grey, blue, black, green, yellow, orange). Assign seasonal membership from the temporal abundance pattern of member taxa (e.g. blue=autumn with n=81 taxa, grey=winter with n=114 taxa; orange/yellow/green are spring/summer). Validate by checking that richness observations separate the spring/summer subnetworks (orange, yellow, green) from the autumn/winter subnetworks (grey, blue, black) and that the six subnetworks match Fig. 1 before writing subnetwork_members.csv.

## Related tools

- **WGCNA (R package)** (Builds the taxon-by-taxon correlation network with soft-thresholding power beta=22, computes the topological overlap dissimilarity matrix, and supports Dynamic Tree Cut module detection on the eDNA abundance indices) — github.com/marinebon/eDNA_microbes_whales
- **R** (Execution environment for the WGCNA pipeline and for writing the module membership table subnetwork_members.csv) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
library(WGCNA); net <- blockwiseModules(t(abundance_index), power = 22, TOMType = "unsigned", deepSplit = 4); write.csv(net$colors, "subnetwork_members.csv")
```

## Evaluation signals

- Dendrogram cut with Dynamic Tree Cut resolves exactly six modules matching Fig. 1
- Module color-to-season assignment reproduces the reported membership (e.g. blue=autumn n=81 taxa, grey=winter n=114 taxa)
- eDNA richness observations distinguish spring/summer subnetworks (orange, yellow, green) from autumn/winter subnetworks (grey, blue, black)
- subnetwork_members.csv contains assignments for all 663 taxonomic groups with no unassigned taxa outside the six modules
- Module member taxa show coherent temporal abundance patterns consistent with their assigned season

## Limitations

- Whether taxon co-occurrence in a module reflects direct predator–prey relationships or a common response to an external driver is unclear
- Network connectivity does not necessarily indicate interaction
- Amplification is subject to primer biases and is affected by inhibitors and variable template abundances, which likely affect the number of taxa detected and thus module membership

## Evidence

- [results] The eDNA richness observations distinguish the spring/summer subnetworks (orange, yellow, green) from the autumn/winter subnetworks (grey, blue, black): [evidence span withheld — anchor status: not_found]
- [discussion] subnetworks that fluctuate in composition and abundance over time: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [abstract] 663 taxonomic groups ranging from microorganisms to mammals: "The resulting dataset encompasses 663 taxonomic groups (at Family or higher taxonomic rank) ranging from microorganisms to mammals."
- [abstract] seasonal shifts across all domains of life: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [readme] readme repository description: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
