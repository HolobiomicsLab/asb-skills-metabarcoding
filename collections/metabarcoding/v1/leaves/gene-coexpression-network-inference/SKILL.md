---
name: gene-coexpression-network-inference
description: Use when a time-series abundance table (taxon-by-sample) should be partitioned
  into correlated subnetworks/modules whose membership reflects seasonal or temporal
  community structure.
when_to_use_negative:
- Input data are raw sequencing reads or quality-filtered FASTQ rather than a normalized
  abundance index table — bioinformatics and quality filtering must occur first
- The data lack a temporal dimension, so module membership cannot be linked to seasonal
  abundance patterns
- Fewer taxa or samples than WGCNA requires for stable correlation and topological
  overlap estimation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: WGCNA (R package)
  role: Builds the taxon-by-taxon weighted correlation structure, topological overlap
    matrix, hierarchical clustering, and Dynamic Tree Cut module detection with soft-thresholding
    power beta=22
  repo: github.com/marinebon/eDNA_microbes_whales
- name: R
  role: Runtime environment for loading the abundance index table, running WGCNA,
    and writing subnetwork_members.csv
  repo: github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:1e8e193306b2cd5981aa6d541e5ce92a22c5caf04705d050642157103f19abb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  - WGCNA (R package)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# gene-coexpression-network-inference

## Summary

Infers co-abundance/co-occurrence subnetworks (modules) from eDNA amplicon-index time-series using weighted gene correlation network analysis (WGCNA), assigning each taxon to a seasonal community subnetwork. Applied here to 663 taxonomic groups from an 18-month Monterey Bay seawater time-series (2015–2016).

## When to use

Use when a time-series abundance table (taxon-by-sample) should be partitioned into correlated subnetworks/modules whose membership reflects seasonal or temporal community structure.

## When NOT to use

- Input data are raw sequencing reads or quality-filtered FASTQ rather than a normalized abundance index table — bioinformatics and quality filtering must occur first
- The data lack a temporal dimension, so module membership cannot be linked to seasonal abundance patterns
- Fewer taxa or samples than WGCNA requires for stable correlation and topological overlap estimation

## Inputs

- eDNA amplicon abundance index table (663 taxonomic groups, Family or higher, across the 18-month Monterey Bay 2015–2016 seawater time-series)
- Sample metadata / time-series collection dates for seasonal assignment

## Outputs

- subnetwork_members.csv (module membership table mapping each taxon to one of six color-labeled subnetworks)
- Six seasonal subnetworks with color labels (spring/summer: orange, yellow, green; autumn/winter: grey, blue, black)
- Topological overlap dissimilarity matrix and clustering dendrogram

## How to apply

Load the eDNA amplicon abundance index table (663 taxonomic groups across the 18-month Monterey Bay 2015–2016 series) from the marinebon/eDNA_microbes_whales repository. Apply WGCNA with soft-thresholding power beta=22 to the abundance indices to build the taxon-by-taxon correlation structure. Convert correlations to a topological overlap dissimilarity matrix, perform hierarchical clustering, and cut the dendrogram with the Dynamic Tree Cut algorithm to detect the six subnetworks (modules). Label each module a color and assign seasonal membership from the temporal abundance pattern of its members (e.g., blue=autumn n=81 taxa, grey=winter n=114 taxa), then verify the six subnetworks and color–season assignments match Fig. 1 and export the membership table to subnetwork_members.csv.

## Related tools

- **WGCNA (R package)** (Builds the taxon-by-taxon weighted correlation structure, topological overlap matrix, hierarchical clustering, and Dynamic Tree Cut module detection with soft-thresholding power beta=22) — github.com/marinebon/eDNA_microbes_whales
- **R** (Runtime environment for loading the abundance index table, running WGCNA, and writing subnetwork_members.csv) — github.com/marinebon/eDNA_microbes_whales

## Examples

```
library(WGCNA); net = blockwiseModules(t(abund_index), power=22, TOMType="unsigned", deepSplit=2, verbose=0); write.csv(net$colors, "subnetwork_members.csv")
```

## Evaluation signals

- Six subnetworks (modules) are recovered from the dendrogram cut, matching the count reported in Fig. 1
- Color–season assignments match Fig. 1, with spring/summer subnetworks (orange, yellow, green) separated from autumn/winter subnetworks (grey, blue, black)
- Module sizes consistent with reported values (e.g., blue=autumn n=81 taxa, grey=winter n=114 taxa)
- subnetwork_members.csv contains every taxon from the 663-group abundance table assigned to exactly one module color
- eDNA richness observations distinguish the spring/summer subnetworks from the autumn/winter subnetworks, confirming coherent seasonal membership

## Limitations

- Network connectivity does not necessarily indicate interaction — co-occurrence may reflect a common response to an external driver rather than direct predator–prey relationships
- Amplification of DNA is subject to primer biases and inhibitors, affecting the number of taxa detected
- Whether taxon co-occurrence is driven by direct interactions or an external driver is unclear; network-derived hypotheses require further hypothesis-based research and experiments
- The analysis assumes the six-module structure reported for this 18-month time-series; longer time-series are needed to increase power and validity

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time.: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [discussion] Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance: "Although network connectivity does not necessarily indicate interaction, a more connected taxon is more likely to present similar patterns of abundance"
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
