---
name: publication-figure-reproduction
description: Use when a task asks to reproduce a specific figure from an article —
  e.g. rebuilding the plant–arthropod bipartite network for seven plant species and
  67 arthropod families from a metabarcoding matrix and saving it as figure6_bipartite_network.png.
when_to_use_negative:
- The input is raw sequencing reads or a metabarcoding OTU/ASV table that has not
  yet been collapsed to plant-species × arthropod-family detections — reproduce the
  figure only from the final network matrix.
- The task asks to generate a novel network visualization rather than match a published
  figure's content and ordering.
- The network involves more than two trophic levels or non-bipartite interaction data
  that the frame2webs/plotweb two-column web format cannot represent.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: R
  role: Analysis environment in which the plant-species × arthropod-family matrix
    is loaded, summarized for family richness, and the figure is rendered to PNG
- name: bipartite
  role: R package providing frame2webs / plotweb / visweb to construct and draw the
    plant–arthropod-family bipartite network matching Figure 6
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:2b2865386745bd8b60e9533e6751c0fbb46d97944328f0e6de2a73b340e7d5fe
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - bipartite
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# publication-figure-reproduction

## Summary

Reconstructs a published figure (here Figure 6, a plant-species × arthropod-family bipartite network) from the underlying detection/abundance matrix using the R bipartite package, so the rendered PNG can be validated against the diversity pattern reported in the article.

## When to use

Use when a task asks to reproduce a specific figure from an article — e.g. rebuilding the plant–arthropod bipartite network for seven plant species and 67 arthropod families from a metabarcoding matrix and saving it as figure6_bipartite_network.png.

## When NOT to use

- The input is raw sequencing reads or a metabarcoding OTU/ASV table that has not yet been collapsed to plant-species × arthropod-family detections — reproduce the figure only from the final network matrix.
- The task asks to generate a novel network visualization rather than match a published figure's content and ordering.
- The network involves more than two trophic levels or non-bipartite interaction data that the frame2webs/plotweb two-column web format cannot represent.

## Inputs

- Plant-species × arthropod-family detection/abundance matrix derived from flower eDNA metabarcoding results
- Seven sampled plant species (incl. Angelica, Daucus, Echium) as row/column identifiers
- Target figure reference (Figure 6 of the article)

## Outputs

- figure6_bipartite_network.png (rendered bipartite plant–arthropod-family network)
- Per-plant-species arthropod family richness summary used to confirm the diversity ordering

## How to apply

Load the plant-species × arthropod-family detection/abundance matrix derived from the flower eDNA metabarcoding results (16S and COI primer sets) into R. Summarize family-level richness per plant species to confirm all seven plant taxa and the 67 recovered arthropod families are represented before plotting. Build the web with the bipartite package via frame2webs, then draw it with plotweb (or inspect with visweb), ordering the lower trophic level so that plant species with the highest family diversity (Apiaceae umbels: Angelica, Daucus) sit adjacent and the lowest (Echium) is visually distinguishable, matching Figure 6. Save the rendered plot as figure6_bipartite_network.png and verify the file is a valid PNG. Judge success by whether the family-diversity ordering across the seven plant species matches the published pattern, not merely that a figure is produced.

## Related tools

- **R** (Analysis environment in which the plant-species × arthropod-family matrix is loaded, summarized for family richness, and the figure is rendered to PNG)
- **bipartite** (R package providing frame2webs / plotweb / visweb to construct and draw the plant–arthropod-family bipartite network matching Figure 6)

## Examples

```
library(bipartite); web <- frame2webs(plant_family_matrix); plotweb(web$plant_family); dev.copy(png, 'figure6_bipartite_network.png'); dev.off()
```

## Evaluation signals

- figure6_bipartite_network.png exists and is a valid PNG file
- The matrix behind the web contains all seven sampled plant taxa and all 67 recovered arthropod families
- Family-diversity ordering across the seven plant species matches the published pattern: Apiaceae umbels (Angelica, Daucus) highest, Echium lowest
- The plotted web has two trophic levels (plants vs. arthropod families) as in Figure 6
- Recovering ≥135 arthropod species across 14 orders in the underlying detections is consistent with the article's reported eDNA yield

## Limitations

- The article notes the experiment was not exhaustive — an even higher arthropod richness could be obtained — so the reproduced network underrepresents true diversity and any richness-based ordering is specific to these samples
- The figure depends on the derived detection/abundance matrix; discrepancies in that intermediate will propagate into the plot
- Matching the published figure is a qualitative validation (visual ordering), not a statistical one

## Evidence

- [abstract] 67 families claim: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] clustering by plant species: "Arthropod communities clustered together according to plant species."
- [abstract] not exhaustive caveat: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
- [other] bipartite workflow from source card: [evidence span withheld — anchor status: not_found]
- [other] diversity pattern from source card: [evidence span withheld — anchor status: not_found]
