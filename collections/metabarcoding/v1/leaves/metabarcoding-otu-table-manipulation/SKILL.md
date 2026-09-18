---
name: metabarcoding-otu-table-manipulation
description: Use when sample-by-taxon community tables from eDNA metabarcoding (e.g.,
  COI and 16S arthropod OTUs from flower samples) are grouped by host plant species
  and the goal is to judge whether arthropod richness is saturated with current sampling
  effort.
when_to_use_negative:
- The input is raw sequencing reads or per-sequence quality data rather than a sample-by-taxon
  community table, since specaccum requires an OTU/ASV matrix
- The analysis goal is community composition/clustering of arthropod assemblages by
  plant species rather than estimating richness saturation with sampling effort
- Plant species are represented by only one flower sample replicate, so no sample-based
  accumulation (rarefaction) curve can be computed for that species
tools:
- name: vegan
  role: R package whose specaccum(method='exact') computes sample-based rarefaction/accumulation
    of arthropod richness per plant species from the OTU/ASV community matrices
- name: R
  role: Language environment in which the COI/16S tables are loaded, subset per plant
    species, and the multi-panel accumulation figure is assembled
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:569fbf15131d8b37bdc4a1b12c2bce4e3be29ba1001a5453f29cff38c6a35bc1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - vegan
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# metabarcoding-otu-table-manipulation

## Summary

Manipulate eDNA metabarcoding OTU/ASV tables (COI and 16S, species and family levels) from flower samples to compute sample-based species accumulation curves per plant species with vegan::specaccum, testing whether recovered arthropod richness saturates with sampling effort.

## When to use

Use when sample-by-taxon community tables from eDNA metabarcoding (e.g., COI and 16S arthropod OTUs from flower samples) are grouped by host plant species and the goal is to judge whether arthropod richness is saturated with current sampling effort.

## When NOT to use

- The input is raw sequencing reads or per-sequence quality data rather than a sample-by-taxon community table, since specaccum requires an OTU/ASV matrix
- The analysis goal is community composition/clustering of arthropod assemblages by plant species rather than estimating richness saturation with sampling effort
- Plant species are represented by only one flower sample replicate, so no sample-based accumulation (rarefaction) curve can be computed for that species

## Inputs

- COI OTU/ASV count table (samples x arthropod taxa, species level)
- COI OTU/ASV count table collapsed to family level
- 16S OTU/ASV count table (samples x arthropod taxa, species level)
- 16S OTU/ASV count table collapsed to family level
- Flower-sample metadata mapping each eDNA replicate to its plant species

## Outputs

- Per-plant-species species accumulation curves (vegan specaccum objects) for COI and 16S at species and family levels
- Multi-panel accumulation-curve plot saved as figure4_accumulation_curves.png (richness vs. number of flower samples)
- Assessment of whether arthropod richness is saturated or still increasing with sampling effort

## How to apply

Load the COI and 16S OTU/ASV tables at both species and family taxonomic levels together with the flower-sample metadata mapping each replicate to its plant species. In R, subset the samples per plant species so each plant species forms a group of flower-sample replicates (a community matrix) for each marker and taxonomic level. Run vegan::specaccum on each grouped matrix with method='exact' to compute sample-based rarefaction/accumulation of arthropod richness. Assemble the four curve sets (COI and 16S at species and family levels) into a single Figure 4-style multi-panel plot with richness on the y-axis and number of flower samples on the x-axis, then save it as figure4_accumulation_curves.png. The rationale is that curves rising steeply without a plateau indicate non-exhaustive sampling, consistent with the paper's finding that higher arthropod richness could still be obtained with this eDNA approach.

## Related tools

- **vegan** (R package whose specaccum(method='exact') computes sample-based rarefaction/accumulation of arthropod richness per plant species from the OTU/ASV community matrices)
- **R** (Language environment in which the COI/16S tables are loaded, subset per plant species, and the multi-panel accumulation figure is assembled)

## Examples

```
sa <- vegan::specaccum(otu_matrix, method='exact'); plot(sa, xlab='Number of flower samples', ylab='Arthropod richness')
```

## Evaluation signals

- Plotted accumulation curves rise steeply and do not reach a plateau, consistent with the paper's finding that sampling was not exhaustive
- Accumulation curves are computed for all four combinations: COI and 16S markers at species and family taxonomic levels
- Each curve is built from flower-sample replicates grouped by a single plant species, with richness on the y-axis and number of flower samples on the x-axis
- specaccum is run with method='exact' (sample-based rarefaction), not random or collector methods
- Final figure is written as figure4_accumulation_curves.png in a Figure 4-style multi-panel layout

## Limitations

- The result is a statement about non-saturation: it indicates greater sampling effort would probably increase recovered arthropod diversity, but does not estimate how much richness remains undiscovered
- Curve shape depends on marker (16S vs COI) and taxonomic level (species vs family), so conclusions should be checked across all four panel combinations rather than from a single curve
- Plant species with too few flower-sample replicates yield unreliable or uncomputable accumulation curves
- The exact accumulation method assumes the observed OTU/ASV table is an accurate sample-by-taxon community matrix; upstream metabarcoding biases are not addressed by this skill

## Evidence

- [other] Species accumulation curves for flower-sample replicates of individual plant species indicated that greater sampling effort would probably increase recovered arthropod diversity, i.e., richness was not saturated.: [evidence span withheld — anchor status: not_found]
- [abstract] Non-exhaustive sampling finding: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
- [abstract] Markers and data origin: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
