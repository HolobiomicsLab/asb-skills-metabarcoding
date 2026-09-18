---
name: arthropod-community-multivariate-modeling
description: Use when you have an OTU/ASV community matrix from eDNA metabarcoding
  (e.g., COI arthropod reads from flower samples) plus sample metadata with plant
  species identity and site coordinates, and need to test or visualize community separation
  by a categorical factor while partialling out spatial
when_to_use_negative:
- Input is raw sequencing reads rather than a composited OTU/ASV matrix — metabarcoding
  preprocessing must happen first
- The question concerns taxonomic richness or presence/absence inventory (e.g., how
  many arthropod species are recovered) rather than community compositional structure
- Metadata lack site coordinates, so spatial covariates cannot be conditioned out
  and an unconstrained or differently constrained ordination is required
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: vegan
  role: R package providing rda (partial redundancy analysis with Condition() for
    lat/lon), anova.cca permutation testing, and plot.cca/ordiplot for site-score
    visualization
- name: R
  role: Analysis environment in which the community matrix, metadata, and vegan ordination
    are scripted and the ordination figure is saved
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:9322f860157435e540101ccb8d26504f99a875d1bdbdd9e9bba540a016c070e7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# arthropod-community-multivariate-modeling

## Summary

Model arthropod eDNA community composition from flower metabarcoding data using partial redundancy analysis (vegan::rda) with site latitude/longitude as conditioning variables, testing whether communities separate by plant species. This reproduces the article's finding that arthropod communities recovered from wild flowers cluster according to plant species, even for interspersed transect samples.

## When to use

Use when you have an OTU/ASV community matrix from eDNA metabarcoding (e.g., COI arthropod reads from flower samples) plus sample metadata with plant species identity and site coordinates, and need to test or visualize community separation by a categorical factor while partialling out spatial covariates.

## When NOT to use

- Input is raw sequencing reads rather than a composited OTU/ASV matrix — metabarcoding preprocessing must happen first
- The question concerns taxonomic richness or presence/absence inventory (e.g., how many arthropod species are recovered) rather than community compositional structure
- Metadata lack site coordinates, so spatial covariates cannot be conditioned out and an unconstrained or differently constrained ordination is required

## Inputs

- COI arthropod OTU/ASV community count matrix (samples × taxa) from flower eDNA metabarcoding
- Sample metadata table with plant species identity and latitude/longitude of sample sites
- Indicator of which samples belong to the interspersed Centaurea/Daucus transect

## Outputs

- Fitted partial RDA model object (vegan 'rda' result with Condition(lat, lon))
- Permutation test result from anova.cca for the constrained plant-species effect
- Ordination plot of site scores colored by plant species (figure5_rda_ordination.png)

## How to apply

Load the COI arthropod community (OTU/ASV) count matrix and the sample metadata table containing plant species identity and latitude/longitude into R. Fit a constrained ordination with vegan::rda using the formula interface so that latitude and longitude enter as conditioning variables via Condition(), making the plant-species effect the only constrained axis set (partial RDA). Assess the significance of the constrained plant-species effect with vegan::anova.cca permutation testing rather than relying on visual inspection alone. Plot site scores with vegan's plot.cca/ordiplot, coloring and labeling samples by plant species, and explicitly mark the interspersed Centaurea and Daucus transect samples to check that they still group by species despite being collected in a shared transect. Save the resulting ordination as figure5_rda_ordination.png.

## Related tools

- **vegan** (R package providing rda (partial redundancy analysis with Condition() for lat/lon), anova.cca permutation testing, and plot.cca/ordiplot for site-score visualization)
- **R** (Analysis environment in which the community matrix, metadata, and vegan ordination are scripted and the ordination figure is saved)

## Examples

```
library(vegan); mod <- rda(otu_matrix ~ plant_species + Condition(latitude, longitude), data = meta); anova(mod); plot(mod, type = 'n'); points(mod, col = meta$plant_species)
```

## Evaluation signals

- The ordination visually reproduces the Figure 5 pattern of arthropod communities clustering by plant species
- The interspersed Centaurea and Daucus transect samples still group somewhat by species within the shared transect area
- The constrained plant-species effect is supported by a significant anova.cca permutation test
- Latitude and longitude appear only as conditioning variables (Condition()), so constrained axes reflect plant species rather than spatial gradients
- The ordination figure is saved as figure5_rda_ordination.png

## Limitations

- The article notes the experiment was not exhaustive and higher arthropod richness could be obtained with this eDNA approach, so community completeness limits ordination resolution
- Separation by plant species was 'some' separation rather than complete — interspersed Centaurea/Daucus samples grouped only 'somewhat' by species
- Recovery was based on two generic primer sets targeting mitochondrial 16S rRNA and COI; primer choice shapes which arthropod taxa enter the community matrix

## Evidence

- [abstract] Communities cluster by plant species under partial RDA: "Arthropod communities clustered together according to plant species."
- [abstract] Data provenance: flower eDNA metabarcoding with 16S/COI primers: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] Taxonomic breadth recovered from flower eDNA: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders"
