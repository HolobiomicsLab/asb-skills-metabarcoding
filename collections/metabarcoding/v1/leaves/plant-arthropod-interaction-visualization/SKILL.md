---
name: plant-arthropod-interaction-visualization
description: Use when you have a plant-species by arthropod-family abundance/detection
  matrix from flower eDNA metabarcoding and need to visualize which plant taxa host
  the highest and lowest arthropod family diversity in a bipartite web.
when_to_use_negative:
- Input is raw sequencing reads (FASTQ) or an OTU/ASV table that has not yet been
  aggregated to arthropod-family level per plant species
- The goal is statistical significance testing of network structure or modularity
  rather than reproducing the descriptive visualization of family distribution
- Interaction data are quantitative time-series or individual-based rather than a
  simple species-by-family detection matrix
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_3053
tools:
- name: R bipartite package (frame2webs / plotweb / visweb)
  role: converts the plant-species × arthropod-family matrix into a two-tier web object
    and draws the bipartite network plot matching Figure 6
- name: R
  role: host environment for loading the metabarcoding-derived matrix, summarizing
    family richness per plant species, and rendering/saving the figure
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:ca709d40878c41311b1aa21fdf072dab4598fdc8c444667f83e10733f9c7a7a2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R bipartite package (frame2webs / plotweb / visweb)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# plant-arthropod-interaction-visualization

## Summary

Construct and render a bipartite plant–arthropod-family interaction network from a plant-species × arthropod-family detection matrix using the R bipartite package, reproducing the figure that shows how eDNA-recovered arthropod families (67 families, 14 orders) are distributed across seven sampled plant species.

## When to use

Use when you have a plant-species by arthropod-family abundance/detection matrix from flower eDNA metabarcoding and need to visualize which plant taxa host the highest and lowest arthropod family diversity in a bipartite web.

## When NOT to use

- Input is raw sequencing reads (FASTQ) or an OTU/ASV table that has not yet been aggregated to arthropod-family level per plant species
- The goal is statistical significance testing of network structure or modularity rather than reproducing the descriptive visualization of family distribution
- Interaction data are quantitative time-series or individual-based rather than a simple species-by-family detection matrix

## Inputs

- plant-species × arthropod-family detection/abundance matrix derived from flower eDNA metabarcoding results (7 plant species × 67 arthropod families)
- metadata mapping each flower sample to its plant species and sampling type (umbel vs. single flower)

## Outputs

- rendered bipartite network figure (e.g. figure6_bipartite_network.png) with plants on one tier and arthropod families on the other
- per-plant-species arthropod family richness summary used to verify the diversity ordering

## How to apply

Load the plant-species × arthropod-family detection/abundance matrix derived from the 16S rRNA and COI metabarcoding results into R and confirm it represents the seven plant taxa and 67 recovered arthropod families. Convert the edge list to a web object with frame2webs and draw it with plotweb (or inspect it with visweb) from the R bipartite package. Order and format the web so that diversity contrasts are visually apparent — in the published pattern the large-surface-area Apiaceae umbels (Angelica, Daucus) show the highest family diversity while single-flower samples (Echium) show the lowest, matching Figure 6 of the article. Save the rendered network as a PNG (e.g. figure6_bipartite_network.png) and check that the diversity ordering across the seven plant species matches the published ranking.

## Related tools

- **R bipartite package (frame2webs / plotweb / visweb)** (converts the plant-species × arthropod-family matrix into a two-tier web object and draws the bipartite network plot matching Figure 6)
- **R** (host environment for loading the metabarcoding-derived matrix, summarizing family richness per plant species, and rendering/saving the figure)

## Examples

```
library(bipartite); web <- frame2webs(edge_list); plotweb(web, method='normal'); dev.copy(png,'figure6_bipartite_network.png'); dev.off()
```

## Evaluation signals

- Output file figure6_bipartite_network.png exists and is a valid PNG image
- The web object contains exactly seven plant taxa and 67 arthropod families as recovered in the study
- Diversity ordering across plant species matches the published pattern: Apiaceae umbels (Angelica, Daucus) highest, Echium (single-flower samples) lowest
- The rendered plot visually separates plants and arthropod families into two tiers with links representing detections
- Recovered families span the reported 14 orders and ecological groups (pollinators, parasitoids, gall inducers, predators, phytophagous species)

## Limitations

- The experiment was not exhaustive; an even higher arthropod richness could be obtained with the flower eDNA approach, so the plotted network underrepresents true diversity
- The figure is descriptive — family diversity per plant species may reflect sampling method (umbel vs. single flower, flower surface area) rather than interaction intensity alone
- Network depends on the metabarcoding-derived detection matrix; primer set choice (16S rRNA vs. COI) affects which families are recovered

## Evidence

- [abstract] 67 families and 14 orders recovered: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders"
- [abstract] communities cluster by plant species: "Arthropod communities clustered together according to plant species."
- [abstract] primer sets used to build the matrix: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] potential of wild flowers as eDNA source: "Here, we test the potential of wild flowers as a novel source of arthropod eDNA."
