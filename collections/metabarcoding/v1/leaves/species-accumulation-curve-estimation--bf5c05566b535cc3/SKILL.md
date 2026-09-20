---
name: species-accumulation-curve-estimation
description: Use when sample-based richness from eDNA metabarcoding (COI/16S OTU or
  ASV tables grouped by plant species or other replicate grouping) must be tested
  for saturation, e.g. asking whether more samples would likely recover additional
  taxa.
when_to_use_negative:
- The goal is inference of taxon presence/absence or community composition rather
  than assessment of sampling-effort saturation
- Sample counts per group are too small (very few replicates per plant species) for
  the exact accumulation method to produce a meaningful rarefaction curve
- The input is raw sequencing reads rather than a processed OTU/ASV community table
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3053
tools:
- name: vegan
  role: R package used to compute sample-based rarefaction/accumulation of arthropod
    richness via specaccum(method='exact') on per-plant-species community matrices
- name: R
  role: Analysis environment in which OTU/ASV tables and flower-sample metadata are
    loaded, subset per plant species, and accumulation curves are plotted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:b879c57bcbee92311da7d607161ca7bb11c6477ece7ffd3bd5fa854082c2ba05
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# species-accumulation-curve-estimation

## Summary

Computes sample-based rarefaction/accumulation curves of arthropod eDNA richness (OTU/ASV or family counts) per plant species using vegan::specaccum with the exact method, to judge whether flower-sample sampling effort has saturated recovered diversity. The paper used this to show higher arthropod richness could be obtained from flower eDNA metabarcoding.

## When to use

Use when sample-based richness from eDNA metabarcoding (COI/16S OTU or ASV tables grouped by plant species or other replicate grouping) must be tested for saturation, e.g. asking whether more samples would likely recover additional taxa.

## When NOT to use

- The goal is inference of taxon presence/absence or community composition rather than assessment of sampling-effort saturation
- Sample counts per group are too small (very few replicates per plant species) for the exact accumulation method to produce a meaningful rarefaction curve
- The input is raw sequencing reads rather than a processed OTU/ASV community table

## Inputs

- COI OTU/ASV count table at species-level taxonomy
- 16S OTU/ASV count table at species-level and family-level taxonomy
- Flower-sample metadata mapping each replicate to its plant species

## Outputs

- Per-plant-species species accumulation curves (richness vs. number of flower samples) for COI and 16S at species and family levels
- Figure 4-style multi-panel accumulation curve plot (e.g. figure4_accumulation_curves.png)

## How to apply

Load the COI and 16S OTU/ASV tables at both species and family taxonomic levels together with the flower-sample metadata mapping each replicate to its plant species. In R, subset the community matrix so each plant species forms a group of flower-sample replicates per marker and taxonomic level. Run vegan::specaccum on each grouped community matrix with method='exact' to compute sample-based rarefaction of arthropod richness. Assemble the resulting curves (species and family levels for both COI and 16S) into a multi-panel figure with richness on the y-axis and number of flower samples on the x-axis, as in the paper's Figure 4-style plot. The decision point is curve shape: steeply rising curves without a plateau indicate sampling was not exhaustive and greater effort would probably increase recovered diversity.

## Related tools

- **vegan** (R package used to compute sample-based rarefaction/accumulation of arthropod richness via specaccum(method='exact') on per-plant-species community matrices)
- **R** (Analysis environment in which OTU/ASV tables and flower-sample metadata are loaded, subset per plant species, and accumulation curves are plotted)

## Examples

```
R: sa <- vegan::specaccum(otu_matrix, method = "exact"); plot(sa, xlab = "Number of flower samples", ylab = "Arthropod richness")
```

## Evaluation signals

- Each curve is computed with vegan::specaccum(method='exact') on a community matrix whose rows/columns match one plant species' flower replicates per marker and taxonomic level
- The multi-panel plot contains accumulation curves for both COI and 16S at species and family taxonomic levels
- Plotted curves rise steeply and do not reach a plateau, consistent with the paper's finding that sampling was not exhaustive
- The x-axis counts flower samples (sampling units) and the y-axis reports arthropod richness (OTUs/ASVs or families)

## Limitations

- Non-plateauing curves indicate only that more taxa would likely be recovered with greater effort; they do not estimate a final richness total
- Curves depend on the processed OTU/ASV tables and the choice of species vs. family taxonomic level, so results can differ between COI and 16S markers
- Curves are computed per plant species, so plant species with few flower-sample replicates yield poorly constrained accumulation curves

## Evidence

- [abstract] The experiment was not exhaustive and higher arthropod richness could be obtained with this eDNA approach: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
- [abstract] eDNA metabarcoding of flowers from several different plant species using two primer sets targeting mitochondrial 16S rRNA and COI: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] eDNA from at least 135 arthropod species in 67 families and 14 orders was recovered: [evidence span withheld — anchor status: not_found]
