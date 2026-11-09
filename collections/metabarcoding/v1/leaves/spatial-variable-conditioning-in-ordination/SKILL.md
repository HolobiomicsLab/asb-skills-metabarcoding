---
name: spatial-variable-conditioning-in-ordination
description: Use when an OTU/ASV community matrix has sample metadata with both a
  categorical grouping factor (plant species) and continuous spatial covariates (lat/lon),
  and you must test whether groups separate after removing spatial structure.
when_to_use_negative:
- The analysis goal is univariate comparison of taxon abundances across plant species
  rather than constrained multivariate ordination
- Sample metadata lacks spatial coordinates, so no conditioning variables can be partialled
  out and a simple unconstrained ordination (e.g. PCA/PCoA/NMDS) suffices
- The community matrix comes from the 16S rRNA primers and the question concerns a
  different marker workflow; this card was grounded on the COI arthropod community
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Statistical environment in which the COI community matrix and metadata are
    loaded and the partial RDA is fitted
- name: vegan
  role: Provides rda with Condition() for partial ordination, anova.cca permutation
    testing, and plot.cca/ordiplot for site-score visualization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:cb996c06c5b777f52f597eac5f3138fcae0655dccaea95d22481f75afebdcce2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# spatial-variable-conditioning-in-ordination

## Summary

A partial RDA approach (vegan) that ordains arthropod eDNA community matrices from wild flowers while conditioning on site latitude and longitude, so that remaining constrained variation reflects plant-species identity (e.g. the interspersed Centaurea/Daucus transect).

## When to use

Use when an OTU/ASV community matrix has sample metadata with both a categorical grouping factor (plant species) and continuous spatial covariates (lat/lon), and you must test whether groups separate after removing spatial structure.

## When NOT to use

- The analysis goal is univariate comparison of taxon abundances across plant species rather than constrained multivariate ordination
- Sample metadata lacks spatial coordinates, so no conditioning variables can be partialled out and a simple unconstrained ordination (e.g. PCA/PCoA/NMDS) suffices
- The community matrix comes from the 16S rRNA primers and the question concerns a different marker workflow; this card was grounded on the COI arthropod community

## Inputs

- COI arthropod OTU/ASV count matrix (samples x taxa)
- Sample metadata table with plant species identity and latitude/longitude of collection sites
- Identification of interspersed Centaurea and Daucus transect samples

## Outputs

- Fitted partial RDA model object (vegan rda with Condition() terms)
- Permutation test result (anova.cca) for the constrained plant-species effect
- Ordination plot of site scores colored by plant species (figure5_rda_ordination.png)

## How to apply

Load the COI arthropod OTU/ASV matrix and sample metadata (plant species identity plus latitude/longitude) into R. Build a partial RDA with vegan's formula interface, e.g. rda(OTU ~ plant_species + Condition(latitude, longitude)), so spatial coordinates are partialled out before the plant-species effect is constrained. Assess significance of the constrained plant-species axis with vegan::anova.cca permutation testing. Plot site scores with plot.cca/ordiplot, coloring and labeling samples by plant species, and specifically mark the interspersed Centaurea and Daucus transect samples to check that co-sampled flowers still group by species. Save the ordination (e.g. figure5_rda_ordination.png) once clustering by plant species is visually reproduced and the permutation test is significant.

## Related tools

- **R** (Statistical environment in which the COI community matrix and metadata are loaded and the partial RDA is fitted)
- **vegan** (Provides rda with Condition() for partial ordination, anova.cca permutation testing, and plot.cca/ordiplot for site-score visualization)

## Examples

```
mod <- rda(otu ~ plant_species + Condition(latitude, longitude), data = meta); anova.cca(mod, permutations = 999); plot(mod)
```

## Evaluation signals

- Site scores in the plot.cca/ordiplot output visibly cluster by plant species, reproducing the Figure 5 pattern
- Interspersed Centaurea and Daucus transect samples group with their own species despite being collected along a shared transect
- vegan::anova.cca returns a significant permutation test for the constrained plant-species effect after latitude/longitude conditioning
- The ordination figure is exported as figure5_rda_ordination.png
- Model formula includes Condition(latitude, longitude), confirming spatial covariates were partialled out before testing species effects

## Limitations

- The article notes the experiment was not exhaustive: eDNA from at least 135 arthropod species was recovered, but higher arthropod richness could be obtained with this eDNA approach, so detected communities are a subset of true flower-visiting assemblages
- Separation by plant species was partial ('some separation'), so the ordination should not be expected to show fully discrete clusters
- Inference depends on eDNA traces left on flowers, which reflect recent arthropod–flower interactions rather than direct abundance surveys

## Evidence

- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] Terrestrial arthropod species leave traces of DNA on the flowers that they interact with: "Our results show that terrestrial arthropod species leave traces of DNA on the flowers that they interact with."
