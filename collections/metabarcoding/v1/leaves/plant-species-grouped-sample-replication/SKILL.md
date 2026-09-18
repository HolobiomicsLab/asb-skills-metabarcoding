---
name: plant-species-grouped-sample-replication
description: Use when flower-sample eDNA metabarcoding replicates must be pooled per
  plant species to build species accumulation curves and judge whether current sampling
  effort exhausts arthropod diversity.
when_to_use_negative:
- The goal is community composition or clustering of arthropod assemblages by plant
  species rather than sampling-effort assessment
- Samples are not flower eDNA replicates traceable to individual plant species, so
  per-species grouping is impossible
- Input is raw sequencing data (FASTQ) rather than an already-built OTU/ASV table
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3050
tools:
- name: vegan
  role: R package whose specaccum function (method='exact') computes sample-based
    rarefaction/accumulation of arthropod richness per plant-species group
- name: R
  role: Environment used to load the OTU/ASV tables, subset samples per plant species,
    run specaccum, and plot the multi-panel figure
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:cc3875fb1e8a318e13d2fd5ba776ab29c750b42386167e0bf7cc5f518c8d2cb5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# plant-species-grouped-sample-replication

## Summary

Groups flower-sample eDNA replicates by plant species and runs sample-based rarefaction (vegan::specaccum, method='exact') on COI and 16S OTU/ASV tables at species and family levels to test whether recovered arthropod richness saturates with sampling effort.

## When to use

Use when flower-sample eDNA metabarcoding replicates must be pooled per plant species to build species accumulation curves and judge whether current sampling effort exhausts arthropod diversity.

## When NOT to use

- The goal is community composition or clustering of arthropod assemblages by plant species rather than sampling-effort assessment
- Samples are not flower eDNA replicates traceable to individual plant species, so per-species grouping is impossible
- Input is raw sequencing data (FASTQ) rather than an already-built OTU/ASV table

## Inputs

- COI OTU/ASV abundance table (species-level and family-level taxonomic assignments)
- 16S OTU/ASV abundance table (species-level and family-level taxonomic assignments)
- Flower-sample metadata mapping each replicate to its plant species

## Outputs

- Per-plant-species exact-method species accumulation curves for each marker (COI, 16S) and taxonomic level (species, family)
- Multi-panel figure saved as figure4_accumulation_curves.png (richness vs. number of flower samples)
- Assessment of whether richness is saturated or greater sampling effort would likely recover higher arthropod diversity

## How to apply

Load the COI and 16S OTU/ASV community tables at both species and family taxonomic levels together with flower-sample metadata mapping each replicate to its plant species. In R, subset the sample set so each plant species forms a group of flower-sample replicates for each marker and taxonomic level, then run vegan::specaccum on each grouped community matrix with method='exact' to compute sample-based rarefaction of arthropod richness. Assemble the resulting accumulation curves (species and family levels for COI and 16S) into a single Figure 4-style multi-panel plot with richness on the y-axis and number of flower samples on the x-axis, and save it as figure4_accumulation_curves.png. The rationale is the paper's finding that arthropod communities cluster by plant species, so per-plant-species replication is the correct grouping for effort-based rarefaction. Validation is judged by whether the curves rise steeply without plateauing, matching the conclusion that the experiment was not exhaustive and higher arthropod richness could be obtained.

## Related tools

- **vegan** (R package whose specaccum function (method='exact') computes sample-based rarefaction/accumulation of arthropod richness per plant-species group)
- **R** (Environment used to load the OTU/ASV tables, subset samples per plant species, run specaccum, and plot the multi-panel figure)

## Examples

```
Rscript -e 'library(vegan); sa <- specaccum(comm_subset, method="exact"); plot(sa, xlab="Number of flower samples", ylab="Richness")'
```

## Evaluation signals

- Each plant species yields its own specaccum object for each combination of marker (COI, 16S) and taxonomic level (species, family)
- Plotted curves show richness rising steeply with number of flower samples and do not reach a plateau
- Figure is saved as figure4_accumulation_curves.png with richness on the y-axis and number of flower samples on the x-axis
- Conclusion drawn from the curves matches the paper's finding that the experiment was not exhaustive and higher richness could be obtained
- Total recovered diversity context matches the article: at least 135 arthropod species in 67 families and 14 orders

## Limitations

- Non-saturating curves indicate insufficient effort but do not quantify how many additional samples are needed
- Rarity estimates depend on the taxonomic assignment level: species-level and family-level tables give different accumulation behaviour
- Interpretation is specific to the eDNA metabarcoding approach with 16S rRNA and COI primers; other markers may recover different richness trajectories

## Evidence

- [other] accumulation curves rise steeply without plateau: [evidence span withheld — anchor status: not_found]
- [abstract] not exhaustive finding: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
- [abstract] clustering by plant species: "Arthropod communities clustered together according to plant species."
- [abstract] two primer sets: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
