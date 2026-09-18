---
name: soft-threshold-scale-free-topology-fitting
description: Use when building a taxon co-abundance network from a time-series abundance
  table (e.g. 663 eDNA taxonomic groups across 18 months of seawater samples) and
  a soft-thresholding power beta must be fit so the correlation structure approximates
  scale-free topology before module detection.
when_to_use_negative:
- Input is raw sequencing reads or quality-filtered FASTQ rather than a taxon-by-sample
  abundance index table — network construction is downstream of bioinformatics and
  quality filtering
- The time series is too short to resolve seasonal community structure, since the
  authors note longer time-series are needed to increase the power and validity of
  eDNA network analyses
- The goal is direct species-level interaction verification — the article states network
  connectivity does not necessarily indicate interaction and co-occurrence may reflect
  a common response to an external driver
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: WGCNA (R package)
  role: Implements soft-thresholding power selection (pickSoftThreshold, beta=22),
    topological overlap computation, and Dynamic Tree Cut module detection for the
    taxon correlation network
  repo: https://github.com/marinebon/eDNA_microbes_whales
- name: R
  role: Runtime environment in which the WGCNA pipeline is executed on the abundance
    index table from the marinebon/eDNA_microbes_whales repository
  repo: https://github.com/marinebon/eDNA_microbes_whales
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-14105-1
    title: Environmental DNA reveals seasonal shifts and potential interactions in
      a marine community
  source_content_hash: sha256:d6df903b7fc3bd1eadf1a0b489d3733cfb3686625bc31644b22ef3e7de14a013
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/marinebon/eDNA_microbes_whales
  tools:
  - R
  - WGCNA (R package)
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# Soft-threshold scale-free topology fitting for co-abundance networks

## Summary

Selects a soft-thresholding power (here beta=22 in WGCNA) so that a taxon-by-taxon correlation network built from eDNA amplicon abundance indices approximates scale-free topology, enabling detection of coherent seasonal subnetworks (modules) from the 18-month Monterey Bay 2015–2016 time series.

## When to use

Use when building a taxon co-abundance network from a time-series abundance table (e.g. 663 eDNA taxonomic groups across 18 months of seawater samples) and a soft-thresholding power beta must be fit so the correlation structure approximates scale-free topology before module detection.

## When NOT to use

- Input is raw sequencing reads or quality-filtered FASTQ rather than a taxon-by-sample abundance index table — network construction is downstream of bioinformatics and quality filtering
- The time series is too short to resolve seasonal community structure, since the authors note longer time-series are needed to increase the power and validity of eDNA network analyses
- The goal is direct species-level interaction verification — the article states network connectivity does not necessarily indicate interaction and co-occurrence may reflect a common response to an external driver

## Inputs

- Taxon-by-sample abundance index table from eDNA multilocus amplicon sequencing (663 taxonomic groups, Family or higher rank, 18-month Monterey Bay 2015–2016 time series, marinebon/eDNA_microbes_whales repository)
- Vector of candidate soft-thresholding powers (e.g. beta = 1–30)
- Network type setting (unsigned taxon correlation network)

## Outputs

- Scale-free topology fit diagnostics (R^2 vs. beta curve) and selected soft-thresholding power (beta=22 in this study)
- Soft-thresholded taxon-by-taxon adjacency / topological overlap dissimilarity matrix
- Hierarchical clustering dendrogram with six color-labeled subnetworks (grey, blue, black, green, yellow, orange)
- Module membership table written to subnetwork_members.csv with seasonal assignments (e.g. blue=autumn n=81 taxa, grey=winter n=114 taxa)

## How to apply

Load the eDNA amplicon abundance index table (663 taxonomic groups, Family or higher rank, from the Monterey Bay 2015–2016 seawater time series) into R and transpose so taxa are the variables. Use the WGCNA R package: run pickSoftThreshold over a power vector (e.g. beta = 1–30) and choose the lowest beta whose scale-free topology fit (R^2) plateau is satisfied — in this study beta=22 was selected for the unsigned taxon-by-taxon correlation network. Apply the chosen beta to soft-threshold the correlations, convert them to a topological overlap dissimilarity matrix, and run hierarchical clustering followed by Dynamic Tree Cut to detect modules. Rationale: soft-thresholding preserves continuous correlation strength while enforcing approximate scale-free connectivity, which the authors used to cluster marine organisms into subnetworks whose member taxa share temporal abundance patterns. Decision point: if the scale-free fit is poor across the power vector, revisit data filtering (e.g. low-variance or sparse taxa) rather than forcing an arbitrary beta.

## Related tools

- **WGCNA (R package)** (Implements soft-thresholding power selection (pickSoftThreshold, beta=22), topological overlap computation, and Dynamic Tree Cut module detection for the taxon correlation network) — https://github.com/marinebon/eDNA_microbes_whales
- **R** (Runtime environment in which the WGCNA pipeline is executed on the abundance index table from the marinebon/eDNA_microbes_whales repository) — https://github.com/marinebon/eDNA_microbes_whales

## Examples

```
Rscript -e 'library(WGCNA); sft <- pickSoftThreshold(t(abundance_table), powerVector=1:30, networkType="unsigned"); beta <- 22; adjacency <- adjacency(t(abundance_table), power=beta)'
```

## Evaluation signals

- pickSoftThreshold diagnostics show the scale-free topology fit R^2 reaching an acceptable plateau at the chosen power (beta=22 for this dataset)
- Dynamic Tree Cut yields exactly six modules, matching the grey, blue, black, green, yellow, orange color labels reported in Fig. 1
- Module sizes match reported values (e.g. blue=autumn with n=81 taxa, grey=winter with n=114 taxa)
- eDNA richness observations separate the spring/summer subnetworks (orange, yellow, green) from the autumn/winter subnetworks (grey, blue, black), confirming coherent seasonal membership
- The final module membership table subnetwork_members.csv exists, contains all 663 taxonomic groups or the filtered subset, and every taxon has exactly one color/season assignment

## Limitations

- Network connectivity does not necessarily indicate interaction; a more connected taxon simply presents more similar abundance patterns
- Whether co-occurrence reflects direct predator-prey relationships or a common response to an external driver is unclear
- Amplification is subject to primer biases, inhibitors, and variable template abundances, which likely affect the number of taxa detected; no method identifies all taxa in an ecosystem
- Taxonomic assignment at species level has lower confidence, constraining the resolution of the abundance index table; longer time-series are needed to increase power and validity

## Evidence

- [abstract] Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life.: "Community network analysis provided evidence of expected predator-prey relationships, trophic linkages, and seasonal shifts across all domains of life."
- [discussion] Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time.: "Our results confirm that groups of marine organisms can be effectively clustered into communities (subnetworks) that fluctuate in composition and abundance over time."
- [readme] This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics.: "This repository is meant for the eDNA group to perform changes and add comments to sequencing data analysis, quality filtering, and bioinformatics."
