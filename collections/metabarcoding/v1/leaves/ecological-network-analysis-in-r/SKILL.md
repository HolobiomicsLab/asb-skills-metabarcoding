---
name: ecological-network-analysis-in-r
description: 'Use when you have a taxon-by-sample detection/abundance matrix (here:
  plant species x arthropod family) and need to build and render a bipartite interaction
  web that makes diversity differences across samples visually apparent.'
when_to_use_negative:
- The data are not a bipartite sample-by-taxon matrix (e.g. raw metabarcoding reads,
  FASTQ files, or ASV tables that have not yet been aggregated to family level)
- The analysis goal is taxonomic composition or ordination (community clustering)
  rather than a two-level interaction-web visualization
edam_operation: http://edamontology.org/operation_0337
tools:
- name: R bipartite package
  role: Provides frame2webs, plotweb and visweb functions to construct and draw the
    bipartite plant-arthropod-family network from the detection matrix
- name: R
  role: Environment for loading the plant-species x arthropod-family matrix, summarizing
    family-level richness, and rendering the network figure
- name: tobiasgf/Bioinformatic-tools
  role: Repository of bioinformatic tools associated with the article's eDNA metabarcoding
    workflow that produces the input matrix
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:9de4a88f00b8c3473aa7a3138d220f2e503e2ae5b329b2f47e397e6a034a1a1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - R
  - R bipartite package
  - tobiasgf/Bioinformatic-tools
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ecological-network-analysis-in-r

## Summary

Construct and draw bipartite ecological networks in R using the bipartite package to visualize how eDNA-recovered taxa (e.g. 67 arthropod families) are distributed across sampling units (e.g. seven plant species), highlighting which units have the highest and lowest partner diversity.

## When to use

Use when you have a taxon-by-sample detection/abundance matrix (here: plant species x arthropod family) and need to build and render a bipartite interaction web that makes diversity differences across samples visually apparent.

## When NOT to use

- The data are not a bipartite sample-by-taxon matrix (e.g. raw metabarcoding reads, FASTQ files, or ASV tables that have not yet been aggregated to family level)
- The analysis goal is taxonomic composition or ordination (community clustering) rather than a two-level interaction-web visualization

## Inputs

- Plant-species x arthropod-family detection/abundance matrix derived from flower eDNA metabarcoding results (7 plant taxa x 67 arthropod families)
- Family-level richness summary per plant species

## Outputs

- Bipartite plant-arthropod-family network figure (e.g. figure6_bipartite_network.png)
- Family-diversity ranking across the seven sampled plant species

## How to apply

Load the plant-species x arthropod-family detection/abundance matrix derived from the flower eDNA metabarcoding results (16S and COI primer sets) into R. Summarize family-level richness per plant species to confirm all seven plant taxa and the 67 recovered arthropod families are represented in the matrix. Build the web with the bipartite package (frame2webs to assemble the matrix into a web object, plotweb or visweb to draw it). Order and format the web so that plant species with the highest family diversity (Apiaceae umbels: Angelica and Daucus, which have large flower surface area) and the lowest (Echium, sampled as single flowers) are visually distinguishable, matching Figure 6 of the article. Render the network to a PNG file and validate the diversity ordering against the published pattern.

## Related tools

- **R bipartite package** (Provides frame2webs, plotweb and visweb functions to construct and draw the bipartite plant-arthropod-family network from the detection matrix)
- **R** (Environment for loading the plant-species x arthropod-family matrix, summarizing family-level richness, and rendering the network figure)
- **tobiasgf/Bioinformatic-tools** (Repository of bioinformatic tools associated with the article's eDNA metabarcoding workflow that produces the input matrix) — tobiasgf/Bioinformatic-tools

## Examples

```
library(bipartite); web <- frame2webs(family_matrix); plotweb(web[[1]])
```

## Evaluation signals

- The rendered figure file (e.g. figure6_bipartite_network.png) exists and is a valid PNG
- The input matrix contains exactly the seven sampled plant taxa and the 67 recovered arthropod families
- The diversity ordering across the seven plant species matches the published pattern: Apiaceae umbels (Angelica, Daucus) highest, Echium lowest
- All arthropod ecological groups present in the data (pollinators, parasitoids, gall inducers, predators, phytophagous species) appear as nodes in the network

## Limitations

- The eDNA experiment was not exhaustive; higher arthropod richness could be obtained with this approach, so the network understates true diversity
- Recovering diverse taxa (135 species in 67 families and 14 orders) from flowers reflects DNA traces left on flowers, not necessarily direct interactions, so edge interpretation requires care

## Evidence

- [abstract] 67 families recovered: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] community structure: "Arthropod communities clustered together according to plant species."
- [results] bipartite plot pattern: [evidence span withheld — anchor status: not_found]
- [methods] network construction: [evidence span withheld — anchor status: not_found]
