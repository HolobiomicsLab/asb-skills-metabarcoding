---
name: bipartite-network-construction
description: Use when a sample-by-taxon detection/abundance matrix (e.g. flower samples
  vs arthropod families from metabarcoding) must be visualized as a bipartite web
  to compare taxon diversity across sample types.
when_to_use_negative:
- Input is raw sequencing reads or OTU/ASV tables that have not yet been collapsed
  to the plant-species x arthropod-family matrix
- The goal is quantitative interaction-strength or modularity analysis rather than
  visualizing which families were recovered on which plant species
- Samples are not paired sample-type vs taxon data (e.g. a single-species time series
  has no bipartite structure to plot)
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R bipartite package (frame2webs / plotweb / visweb)
  role: Constructs the two-level web from the plant x arthropod-family matrix and
    draws the network plot ordered by family diversity per plant species
  repo: https://github.com/tobiasgf/Bioinformatic-tools
- name: R
  role: Runtime environment in which the matrix is loaded, family richness per plant
    species is summarized, and the bipartite figure is rendered and saved
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:fb169ff07aea660a4638cf75c1084858cf6e6a071165c438e0fe908d98d5de46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - R
  - R bipartite package (frame2webs / plotweb / visweb)
  - bipartite
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# bipartite-network-construction

## Summary

Constructs and renders a two-level (plant x arthropod-family) interaction network from a detection/abundance matrix using the R 'bipartite' package, so that differences in family richness across the seven sampled plant species — highest in Apiaceae umbels (Angelica, Daucus), lowest in Echium single flowers — are visually apparent as in Figure 6.

## When to use

Use when a sample-by-taxon detection/abundance matrix (e.g. flower samples vs arthropod families from metabarcoding) must be visualized as a bipartite web to compare taxon diversity across sample types.

## When NOT to use

- Input is raw sequencing reads or OTU/ASV tables that have not yet been collapsed to the plant-species x arthropod-family matrix
- The goal is quantitative interaction-strength or modularity analysis rather than visualizing which families were recovered on which plant species
- Samples are not paired sample-type vs taxon data (e.g. a single-species time series has no bipartite structure to plot)

## Inputs

- Plant-species x arthropod-family detection/abundance matrix derived from flower eDNA metabarcoding results (7 plant species, 67 arthropod families, 135 species, 14 orders)
- Family-level richness table per plant species (for pre-plot verification)

## Outputs

- Rendered bipartite network figure (e.g. figure6_bipartite_network.png) showing plant species and arthropod families with diversity ordering visually apparent

## How to apply

Load the plant-species x arthropod-family detection/abundance matrix derived from the flower eDNA metabarcoding results (two 16S rRNA and COI primer sets, 7 plant taxa, 67 recovered arthropod families) into R. Summarize family-level richness per plant species first to confirm all seven plant taxa and 67 families are represented in the matrix. Build the web with frame2webs and draw it with plotweb (or inspect it with visweb) from the bipartite package, arranging columns so plant species rank by arthropod family diversity. Format the plot so the diversity ordering — Apiaceae umbels (Angelica, Daucus) highest, single-flower samples (Echium) lowest — is visually obvious, matching Figure 6. Render and save the network figure, then validate the ordering against the published pattern before accepting the output.

## Related tools

- **R bipartite package (frame2webs / plotweb / visweb)** (Constructs the two-level web from the plant x arthropod-family matrix and draws the network plot ordered by family diversity per plant species) — https://github.com/tobiasgf/Bioinformatic-tools
- **R** (Runtime environment in which the matrix is loaded, family richness per plant species is summarized, and the bipartite figure is rendered and saved)

## Examples

```
library(bipartite); web <- frame2webs(mat, type='up'); plotweb(web[[1]], high.lablength=0, low.lablength=0)
```

## Evaluation signals

- Output PNG file exists and is a valid PNG image
- The plotted web contains all 7 plant taxa and 67 arthropod families from the metabarcoding matrix
- Family-diversity ordering across plant species matches the published pattern: Apiaceae umbels (Angelica, Daucus) highest, Echium lowest
- Network layout visually separates high-diversity large-surface-area samples (umbels) from low-diversity single-flower samples, as in Figure 6
- Richness summary computed before plotting matches the family counts shown in the plotted web

## Limitations

- The recovered diversity is not exhaustive: the article notes 'an even higher arthropod richness could be obtained using this eDNA approach', so the network understates true family richness
- Interpretation depends on sampling format: large-surface-area samples (Apiaceae umbels) can appear more diverse than single flowers regardless of true visitation, confounding diversity comparisons across plant species
- Matrix construction depends on upstream metabarcoding detection decisions (16S rRNA and COI primer sets, contamination filtering); errors there propagate into the web

## Evidence

- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
