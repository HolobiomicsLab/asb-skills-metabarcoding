---
name: ordination-biplot-visualization
description: Use when a sample-by-OTU/ASV community matrix has a categorical grouping
  factor (e.g. plant species identity) plus site covariates (lat/long) and the goal
  is to visualize and permutation-test whether communities cluster by that factor.
when_to_use_negative:
- The community data have no grouping factor or the grouping is confounded with site
  coordinates such that Condition(lat, lon) would remove the effect of interest
- The input is raw sequencing reads or a per-sample taxon table without metadata linking
  samples to plant species and coordinates
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3050
tools:
- name: vegan
  role: R package used to fit the partial RDA (rda with Condition(lat, lon)), run
    the anova.cca permutation test, and draw the site-score ordination plot
- name: R
  role: Language/environment in which the community matrix, metadata, and vegan ordination
    are handled
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:e9688b5c6df65d5942dc2655c4679b6c54297e3ca8cf5bc0beef1de58aa5342f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# ordination-biplot-visualization

## Summary

Reproduce constrained ordination (partial RDA via vegan::rda) of flower eDNA arthropod community matrices, testing whether communities separate by plant species while conditioning on latitude/longitude of sampling sites, and render a species-colored site-score biplot.

## When to use

Use when a sample-by-OTU/ASV community matrix has a categorical grouping factor (e.g. plant species identity) plus site covariates (lat/long) and the goal is to visualize and permutation-test whether communities cluster by that factor.

## When NOT to use

- The community data have no grouping factor or the grouping is confounded with site coordinates such that Condition(lat, lon) would remove the effect of interest
- The input is raw sequencing reads or a per-sample taxon table without metadata linking samples to plant species and coordinates

## Inputs

- COI arthropod OTU/ASV community matrix (samples x taxa)
- Sample metadata table with plant species identity and latitude/longitude coordinates of sample sites

## Outputs

- Fitted partial RDA model (vegan::rda) with plant species as the constrained factor and lat/lon as conditioning variables
- anova.cca permutation test result (significance of the constrained plant-species effect)
- RDA ordination biplot figure colored/labeled by plant species, e.g. figure5_rda_ordination.png

## How to apply

Load the COI arthropod OTU/ASV matrix and the sample metadata table holding plant species identity and latitude/longitude into R. Build a partial RDA with vegan::rda using the formula interface, placing latitude and longitude inside Condition() so the constrained axis captures the plant-species effect after partialling out spatial position. Assess significance of the constrained effect with vegan::anova.cca permutation testing. Plot site scores with plot.cca/ordiplot, coloring and labeling samples by plant species, and explicitly mark the interspersed Centaurea and Daucus transect samples so that within-transect segregation by species is visible; save the plot (e.g. figure5_rda_ordination.png).

## Related tools

- **vegan** (R package used to fit the partial RDA (rda with Condition(lat, lon)), run the anova.cca permutation test, and draw the site-score ordination plot)
- **R** (Language/environment in which the community matrix, metadata, and vegan ordination are handled)

## Examples

```
r <- rda(otu ~ plant_species + Condition(latitude, longitude), data=meta); anova(r); plot(r, type='n'); points(r, col=meta$plant_species)
```

## Evaluation signals

- The ordination visually reproduces the Figure 5 pattern: arthropod community site scores cluster by plant species
- The constrained plant-species effect is supported by a significant vegan::anova.cca permutation test
- Interspersed Centaurea and Daucus transect samples group somewhat by species within the shared transect area of the plot
- The conditioning variables (latitude, longitude) are confirmed to sit inside Condition() in the rda formula, not as explanatory terms

## Limitations

- The article notes the experiment was not exhaustive and higher arthropod richness could be obtained, so the ordination reflects a partially sampled community
- Separation by plant species was only partial ('some separation'), so visual clustering may be imperfect even when the model is significant

## Evidence

- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
- [abstract] eDNA recovered from diverse arthropod taxa underpinning the community matrix: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders"
- [other] Task workflow defining the partial RDA procedure: [evidence span withheld — anchor status: not_found]
- [other] Interspersed transect samples still grouped by species: [evidence span withheld — anchor status: not_found]
