---
name: multimarker-richness-visualization
description: Use when arthropod eDNA richness from grouped flower-sample replicates
  (per plant species, per marker) must be rarefied and plotted as multi-panel accumulation
  curves to assess sampling sufficiency.
when_to_use_negative:
- The question is community composition/clustering of arthropods by plant species
  rather than richness saturation with sampling effort
- Samples are not grouped flower-sample replicates of individual plant species (e.g.,
  single pooled samples or non-flower eDNA sources), so per-species accumulation cannot
  be computed
- Only a single taxonomic level or single marker is of interest and no multi-panel
  COI/16S comparison is needed — a simpler rarefaction call suffices
edam_operation: http://edamontology.org/operation_2928
tools:
- name: vegan
  role: R package whose specaccum function (method='exact') computes the sample-based
    rarefaction/accumulation curves for each plant-species group
- name: R
  role: Environment used to subset the OTU/ASV tables per plant species and marker,
    run specaccum, and render the multi-panel PNG figure
  repo: https://github.com/tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:83bd39befe939d092b463c60846e378e3cb7404cda450fec28383022b4cc2c04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - R
  - vegan
  edam_operation: http://edamontology.org/operation_2928
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# multimarker-richness-visualization

## Summary

Computes and plots sample-based species accumulation curves (exact-method rarefaction) per plant species from COI and 16S eDNA metabarcoding OTU/ASV tables, at species and family taxonomic levels, to judge whether recovered arthropod richness saturates with flower sampling effort.

## When to use

Use when arthropod eDNA richness from grouped flower-sample replicates (per plant species, per marker) must be rarefied and plotted as multi-panel accumulation curves to assess sampling sufficiency.

## When NOT to use

- The question is community composition/clustering of arthropods by plant species rather than richness saturation with sampling effort
- Samples are not grouped flower-sample replicates of individual plant species (e.g., single pooled samples or non-flower eDNA sources), so per-species accumulation cannot be computed
- Only a single taxonomic level or single marker is of interest and no multi-panel COI/16S comparison is needed — a simpler rarefaction call suffices

## Inputs

- COI OTU/ASV abundance table at species-level taxonomy
- COI OTU/ASV abundance table at family-level taxonomy
- 16S OTU/ASV abundance table at species-level taxonomy
- 16S OTU/ASV abundance table at family-level taxonomy
- Flower-sample metadata mapping each replicate to its plant species

## Outputs

- Per-plant-species exact-method species accumulation curves (vegan specaccum objects) for COI and 16S at species and family levels
- Multi-panel Figure 4-style plot saved as figure4_accumulation_curves.png with richness vs. number of flower samples

## How to apply

Load the COI and 16S OTU/ASV tables at both species and family taxonomic levels together with flower-sample metadata mapping each replicate to its plant species. In R, subset the community matrix into groups of flower-sample replicates per plant species for each marker and taxonomic level. Run vegan::specaccum with method='exact' on each grouped matrix to compute sample-based rarefaction of arthropod richness. Assemble the resulting accumulation curves into a single Figure 4-style multi-panel plot (richness on the y-axis, number of flower samples on the x-axis) and save it as figure4_accumulation_curves.png. The rationale is the article's finding that curves rising steeply without a plateau indicate the experiment was not exhaustive, so greater sampling effort would probably recover higher arthropod diversity.

## Related tools

- **vegan** (R package whose specaccum function (method='exact') computes the sample-based rarefaction/accumulation curves for each plant-species group)
- **R** (Environment used to subset the OTU/ASV tables per plant species and marker, run specaccum, and render the multi-panel PNG figure) — https://github.com/tobiasgf/Bioinformatic-tools

## Examples

```
sp <- vegan::specaccum(otu_mat[meta$plant_species == sp_name, ], method = "exact")
```

## Evaluation signals

- Each plotted curve is built from the correct group: one accumulation curve per plant species per marker (COI/16S) and taxonomic level (species/family)
- specaccum was run with method='exact' (sample-based exact rarefaction), not random or collector methods
- Figure axes follow the Figure 4 layout: richness on y-axis, number of flower samples on x-axis, panels covering COI and 16S at species and family levels
- Output file figure4_accumulation_curves.png exists and renders all panels
- Validation: plotted curves rise steeply and do not reach a plateau, consistent with the paper's finding that the sampling was not exhaustive and higher arthropod richness could be obtained

## Limitations

- Curves that appear to plateau for a given plant species should not be over-interpreted as exhaustive sampling; the paper's conclusion is that, overall, the experiment was not exhaustive
- Richness estimates depend on the recovered diversity of the marker set: eDNA recovered at least 135 arthropod species in 67 families and 14 orders, so accumulation curves reflect only detectable OTU/ASV richness, not true arthropod richness
- The exact-method accumulation requires multiple flower-sample replicates per plant species; groups with very few replicates give unstable curves

## Evidence

- [abstract] non-exhaustive sampling: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
- [abstract] recovered diversity scale: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] markers underlying the curves: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
