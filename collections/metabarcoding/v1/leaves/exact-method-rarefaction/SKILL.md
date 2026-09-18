---
name: exact-method-rarefaction
description: Use when replicate eDNA samples can be grouped (e.g., flower replicates
  per plant species) and you must judge from accumulation curves whether richness
  has plateaued or more sampling would recover additional taxa.
when_to_use_negative:
- The question concerns which arthropod taxa are present or how communities cluster
  by plant species, not whether sampling effort is sufficient
- Samples cannot be grouped into replicate sets per plant species (e.g., singletons
  or heterogeneous environmental samples), so accumulation curves would be uninterpretable
tools:
- name: vegan
  role: R package providing specaccum with method='exact' for sample-based rarefaction
    of arthropod richness per plant species
- name: R
  role: Environment in which OTU/ASV tables are subset per plant species and accumulation
    curves are plotted and saved
  repo: https://github.com/tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:20bab28e682acba7beed084a6e55d87f08d6ca7ef12bb64d73b0b1d33b27ac42
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - R
  - vegan
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# exact-method-rarefaction

## Summary

Sample-based species accumulation (rarefaction) using vegan::specaccum with method='exact' on arthropod eDNA OTU/ASV tables grouped per plant species, to test whether arthropod richness recovered from flower eDNA saturates with sampling effort.

## When to use

Use when replicate eDNA samples can be grouped (e.g., flower replicates per plant species) and you must judge from accumulation curves whether richness has plateaued or more sampling would recover additional taxa.

## When NOT to use

- The question concerns which arthropod taxa are present or how communities cluster by plant species, not whether sampling effort is sufficient
- Samples cannot be grouped into replicate sets per plant species (e.g., singletons or heterogeneous environmental samples), so accumulation curves would be uninterpretable

## Inputs

- COI and 16S OTU/ASV abundance tables at species and family taxonomic levels
- Flower-sample metadata mapping each replicate to its plant species

## Outputs

- Exact-method species accumulation curves (richness vs. number of flower samples) per plant species, marker, and taxonomic level
- Figure 4-style multi-panel plot (e.g., figure4_accumulation_curves.png)

## How to apply

Load the COI and 16S OTU/ASV count tables at species and family taxonomic levels together with flower-sample metadata mapping each replicate to its plant species. Subset the samples per plant species so each species forms a group of flower-sample replicates for each marker and taxonomic level. Run vegan::specaccum on each grouped community matrix with method='exact' to compute exact sample-based rarefaction of arthropod richness, then assemble the four curve sets (COI/16S x species/family) into a multi-panel figure with richness on the y-axis and number of flower samples on the x-axis. The rationale is that curves that rise steeply without plateau indicate the sampling was not exhaustive, matching the paper's conclusion that higher arthropod richness could be obtained with this eDNA approach.

## Related tools

- **vegan** (R package providing specaccum with method='exact' for sample-based rarefaction of arthropod richness per plant species)
- **R** (Environment in which OTU/ASV tables are subset per plant species and accumulation curves are plotted and saved) — https://github.com/tobiasgf/Bioinformatic-tools

## Examples

```
R: sa <- vegan::specaccum(comm, method = "exact"); plot(sa, xlab = "Number of flower samples", ylab = "Arthropod richness")
```

## Evaluation signals

- Curves rise steeply and do not reach a plateau, consistent with the paper's finding that sampling was not exhaustive
- One accumulation curve is produced for each combination of marker (COI, 16S) and taxonomic level (species, family) per plant species
- The x-axis of each curve equals the number of flower samples in the corresponding plant-species group
- The figure is saved as figure4_accumulation_curves.png in a multi-panel layout matching Figure 4 of the paper

## Limitations

- Non-saturating curves indicate effort was insufficient but do not quantify how many additional arthropod species would be recovered
- The README for tobiasgf/Bioinformatic-tools contained no documentation, so implementation details must come from the article workflow

## Evidence

- [abstract] not exhaustive / higher richness: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
- [abstract] markers and data source: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
