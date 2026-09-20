---
name: edna-metabarcoding-community-profiling
description: Use when you have an eDNA metabarcoding community matrix (COI or 16S
  OTU/ASV table) plus sample metadata with host/source identity (e.g. plant species)
  and site coordinates, and want to test whether communities separate by that identity.
when_to_use_negative:
- Input is raw sequencing reads or FASTQ data rather than an already-derived OTU/ASV
  community matrix — bioinformatic processing must precede this ordination step
- The question concerns taxonomic richness or diversity of the flower eDNA rather
  than whether community composition separates by sample group (the article notes
  richness could be even higher and the experiment was not exhaustive)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_3911
tools:
- name: vegan
  role: R package used to run the partial redundancy analysis (rda with Condition()
    for latitude/longitude), permutation testing (anova.cca), and ordination plotting
    (ordiplot/plot.cca)
- name: R
  role: Environment in which the community matrix and metadata are loaded and the
    RDA workflow is executed
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:25dc3dbc0c8a280a3d5614467b3a757ebaf7dcf784f5e954ed6ea3e982498938
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_3911
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# edna-metabarcoding-community-profiling

## Summary

Profile arthropod community composition from flower eDNA metabarcoding data (COI and 16S rRNA OTU/ASV matrices) and test whether communities cluster by plant species, e.g. via redundancy analysis (vegan::rda) with site latitude/longitude as conditioning variables, as in Thomsen & Sigsgaard (2019).

## When to use

Use when you have an eDNA metabarcoding community matrix (COI or 16S OTU/ASV table) plus sample metadata with host/source identity (e.g. plant species) and site coordinates, and want to test whether communities separate by that identity.

## When NOT to use

- Input is raw sequencing reads or FASTQ data rather than an already-derived OTU/ASV community matrix — bioinformatic processing must precede this ordination step
- The question concerns taxonomic richness or diversity of the flower eDNA rather than whether community composition separates by sample group (the article notes richness could be even higher and the experiment was not exhaustive)

## Inputs

- COI arthropod OTU/ASV community matrix (samples x taxa counts)
- Sample metadata table with plant species identity and latitude/longitude of sample sites
- Transect annotation identifying interspersed Centaurea and Daucus samples

## Outputs

- Partial RDA model (vegan) with plant species as the constrained factor and latitude/longitude conditioned
- Permutation test results (anova.cca) for the constrained plant-species effect
- Ordination figure of site scores colored/labeled by plant species (e.g. figure5_rda_ordination.png)

## How to apply

Load the COI arthropod OTU/ASV community matrix and the sample metadata table holding plant species identity plus latitude/longitude of sample sites into R. Run vegan::rda on the community matrix with latitude and longitude supplied as conditioning variables via Condition() in the vegan formula interface, so the constrained axis captures only the plant-species effect. Assess significance of the constrained plant-species effect with vegan::anova.cca permutation testing on the RDA model. Plot site scores with vegan's ordiplot/plot.cca, coloring and labeling samples by plant species; in this study even interspersed Centaurea and Daucus transect samples grouped somewhat by species, which should be marked within the shared transect area. Save the ordination (e.g. figure5_rda_ordination.png) and check that the visual clustering matches the published Figure 5 pattern and that the permutation test supports the constrained model.

## Related tools

- **vegan** (R package used to run the partial redundancy analysis (rda with Condition() for latitude/longitude), permutation testing (anova.cca), and ordination plotting (ordiplot/plot.cca))
- **R** (Environment in which the community matrix and metadata are loaded and the RDA workflow is executed)

## Examples

```
rda(otu ~ Condition(lat) + Condition(lon) + plant_species, data=meta); anova.cca(m); plot(m)
```

## Evaluation signals

- Ordination plot visually reproduces the Figure 5 pattern: arthropod communities cluster by plant species
- vegan::anova.cca permutation test of the constrained plant-species effect is statistically significant
- Interspersed Centaurea and Daucus transect samples show at least partial separation within the shared transect area
- Site scores are plotted after conditioning on latitude/longitude (partial RDA), not from an unconstrained model
- Recovered community spans the expected diversity of ecological groups (pollinators, parasitoids, gall inducers, predators, phytophagous species)

## Limitations

- The experiment was not exhaustive — higher arthropod richness could be obtained with this eDNA approach
- Separation of communities by plant species was only partial ('some separation'), not complete; interspersed Centaurea/Daucus samples grouped 'somewhat' by species
- Ordination depends on the quality of the upstream COI/16S metabarcoding pipeline; the article's repository README provided no additional pipeline documentation in this context

## Evidence

- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
- [abstract] metabarcoding workflow with two primer sets: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] breadth of recovered community: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders"
- [abstract] exhaustiveness caveat: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
