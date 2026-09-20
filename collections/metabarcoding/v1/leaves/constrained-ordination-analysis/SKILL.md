---
name: constrained-ordination-analysis
description: Use when an OTU/ASV community matrix has a categorical metadata variable
  (e.g. plant species) and numeric covariates (e.g. latitude/longitude), and the goal
  is to visualize and permutation-test whether communities cluster by that category.
when_to_use_negative:
- The community matrix is not sample-by-taxon (e.g. only a taxon list or abundance
  summary without matching metadata) — ordination requires the full matrix plus per-sample
  species/covariate annotations.
- The analysis question is unconstrained pattern description of community structure
  without a categorical hypothesis variable — an unconstrained ordination (PCA/PCoA/NMDS)
  is then more appropriate than RDA with Condition().
- Metadata lack geographic covariates or there is no covariate to condition on, making
  a partial RDA (Condition(latitude, longitude)) mismatched to the design.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3050
tools:
- name: R
  role: Runtime environment in which the community matrix and metadata are loaded
    and the RDA, permutation test, and ordination plot are executed
- name: vegan
  role: Provides rda (with Condition() for the partial formula), anova.cca permutation
    testing, and ordiplot/plot.cca for the species-colored site-score figure
  repo: https://github.com/tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:74604e86dac399243fa45dd13b0aa49f05358f4482830ac0243f3ead99032acb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - R
  - vegan
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# constrained-ordination-analysis

## Summary

Applies partial redundancy analysis (RDA) in vegan to eDNA metabarcoding community matrices (e.g. flower-surface arthropod OTU/ASV tables from COI/16S amplicons) to test whether biological communities are constrained by a categorical factor such as plant species, after removing geographic covariates (latitude/longitude) via Condition().

## When to use

Use when an OTU/ASV community matrix has a categorical metadata variable (e.g. plant species) and numeric covariates (e.g. latitude/longitude), and the goal is to visualize and permutation-test whether communities cluster by that category.

## When NOT to use

- The community matrix is not sample-by-taxon (e.g. only a taxon list or abundance summary without matching metadata) — ordination requires the full matrix plus per-sample species/covariate annotations.
- The analysis question is unconstrained pattern description of community structure without a categorical hypothesis variable — an unconstrained ordination (PCA/PCoA/NMDS) is then more appropriate than RDA with Condition().
- Metadata lack geographic covariates or there is no covariate to condition on, making a partial RDA (Condition(latitude, longitude)) mismatched to the design.

## Inputs

- OTU/ASV count matrix (samples x taxa) from eDNA metabarcoding of flower eDNA (COI and/or 16S amplicons)
- Sample metadata table with plant species identity and site latitude/longitude coordinates

## Outputs

- Fitted partial RDA model object (vegan rda/cca)
- Permutation test p-value for the constrained plant-species effect (anova.cca)
- Ordination biplot/site-score figure colored by plant species (e.g. figure5_rda_ordination.png)

## How to apply

Load the COI (or 16S) arthropod OTU/ASV matrix and the sample metadata table (plant species identity, site latitude/longitude) into R. Run vegan::rda with the formula interface, supplying latitude and longitude inside Condition() so the partial RDA removes geographic position before testing the constrained plant-species effect. Assess significance of the constrained effect with vegan::anova.cca permutation testing on the fitted model. Plot site scores with ordiplot/plot.cca, coloring and labeling samples by plant species (in the Thomsen & Sigsgaard 2019 case, marking the interspersed Centaurea and Daucus transect samples within the shared transect area), and export the ordination figure (e.g. figure5_rda_ordination.png). The rationale is that flower eDNA leaves traces from interacting arthropods, so community composition should track the plant species if the metabarcoding (COI/16S primers) captured the ecological signal.

## Related tools

- **R** (Runtime environment in which the community matrix and metadata are loaded and the RDA, permutation test, and ordination plot are executed)
- **vegan** (Provides rda (with Condition() for the partial formula), anova.cca permutation testing, and ordiplot/plot.cca for the species-colored site-score figure) — https://github.com/tobiasgf/Bioinformatic-tools

## Examples

```
mod <- rda(otu ~ plant_species + Condition(latitude, longitude), data=meta); anova(mod); plot(mod, type='points')
```

## Evaluation signals

- The ordination figure shows arthropod communities clustering/separating according to plant species, reproducing the Figure 5 pattern
- The interspersed Centaurea and Daucus transect samples still group somewhat by species within the shared transect area
- anova.cca permutation test on the constrained plant-species effect returns a significant result
- Condition(latitude, longitude) terms are present in the model formula, confirming geographic covariates were partialled out
- Output figure file (figure5_rda_ordination.png) exists and site scores are colored/labeled by plant species

## Limitations

- The article notes the experiment was not exhaustive and higher arthropod richness could be obtained with this eDNA approach, so the community matrix may undersample true richness.
- Separation by plant species was only partial ('some separation'), not a clean segregation of all species; interspersed transect samples grouped only 'somewhat' by species.

## Evidence

- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
- [abstract] Flowers as novel eDNA source: "Our results show that terrestrial arthropod species leave traces of DNA on the flowers that they interact with."
- [abstract] Metabarcoding primers: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] RDA workflow: [evidence span withheld — anchor status: not_found]
