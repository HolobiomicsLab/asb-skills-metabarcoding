---
name: microbiome-sample-grouping-by-hypervariable-region
description: Use when genus-level collapsed ASV abundance tables (e.g., Greengenes-assigned,
  QIIME 2-produced) must be grouped by 16S hypervariable region to assess compositional
  dissimilarity in ordination space between region combinations.
when_to_use_negative:
- Input is raw reads or ASVs not yet collapsed at the genus level with Greengenes
  assignments — collapse taxonomy upstream first
- The goal is comparison against full-length 16S rRNA sequencing, which the article
  explicitly did not perform
- The abundance table is not compositional microbiome count data (Bray–Curtis on genus-level
  relative abundances is the assumption here)
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: vegan
  role: Computes the Bray–Curtis dissimilarity matrix and runs metaMDS NMDS ordination
- name: phyloseq
  role: Handles the genus-level collapsed ASV abundance table and sample region-grouping
    metadata
- name: ggplot2
  role: Plots the NMDS ordination with region-group color coding and 0.95 confidence
    ellipses, plus the stress plot
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:6ad3ceb3c01e8ab56eed7d3010b2663ecd39155e65599f67ec7af6e7245570dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - phyloseq
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# microbiome-sample-grouping-by-hypervariable-region

## Summary

Groups sputum microbiome samples by 16S rRNA hypervariable region combination (V1–V2, V3–V4, V5–V7, V7–V9) and tests whether they differ compositionally via NMDS ordination of Bray–Curtis dissimilarities, as used in the López-Aladid et al. 2023 Scientific Reports study of respiratory samples.

## When to use

Use when genus-level collapsed ASV abundance tables (e.g., Greengenes-assigned, QIIME 2-produced) must be grouped by 16S hypervariable region to assess compositional dissimilarity in ordination space between region combinations.

## When NOT to use

- Input is raw reads or ASVs not yet collapsed at the genus level with Greengenes assignments — collapse taxonomy upstream first
- The goal is comparison against full-length 16S rRNA sequencing, which the article explicitly did not perform
- The abundance table is not compositional microbiome count data (Bray–Curtis on genus-level relative abundances is the assumption here)

## Inputs

- Genus-level collapsed ASV abundance table (Greengenes assignments, produced upstream in QIIME 2)
- Sample-to-hypervariable-region grouping labels (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- Bray–Curtis dissimilarity matrix between samples
- NMDS ordination figure colored by region group with 0.95 confidence ellipses (analog of Figure 5)
- NMDS residual stress diagnostic plot (analog of supplementary S2)
- Stress value (between 0 and 1) and fit statistics (R2, pAdonis)

## How to apply

Load the genus-level collapsed ASV abundance table (taxonomic assignments from the Greengenes database, collapsed at genus level in QIIME 2) together with the sample-to-region grouping labels (V1–V2, V3–V4, V5–V7, V7–V9). Compute the Bray–Curtis dissimilarity matrix between samples using the vegan package, then run NMDS ordination with vegan's metaMDS and record the final stress value and fit statistics (in the article: R2 = 0.44, pAdonis < 0.001). Plot the NMDS1/NMDS2 solution with ggplot2, color-coding samples by hypervariable region group and drawing 0.95 confidence ellipses around each group. Generate a residual stress plot as a diagnostic, mirroring supplementary material S2. Decision point: interpret groups as compositionally similar when their 0.95 ellipses overlap in both NMDS1 and NMDS2 (as reported for V3–V4 and V5–V7).

## Related tools

- **vegan** (Computes the Bray–Curtis dissimilarity matrix and runs metaMDS NMDS ordination)
- **phyloseq** (Handles the genus-level collapsed ASV abundance table and sample region-grouping metadata)
- **ggplot2** (Plots the NMDS ordination with region-group color coding and 0.95 confidence ellipses, plus the stress plot)

## Examples

```
Rscript -e 'ord <- vegan::metaMDS(vegan::vegdist(genus_counts, method="bray"), k=2); plot(ord, type="n"); vegan::ordiellipse(ord, region_groups, draw="polygon", conf=0.95)'
```

## Evaluation signals

- Recorded NMDS stress value falls between 0 and 1
- Fit statistics reported, consistent with the article's R2 = 0.44 and pAdonis < 0.001
- Output figure reproduces the reported pattern: V3–V4 and V5–V7 ellipses overlap in both NMDS1 and NMDS2, while V1–V2 and V7–V9 separate
- 0.95 confidence ellipses are drawn around all region groups
- Residual stress plot exists as a diagnostic, mirroring supplementary material S2

## Limitations

- Reference database mismatch: regions were trained with outdated Greengenes instead of more updated SILVA versions
- Illumina MiSeq 300 bp length limit constrains 16S rRNA profiling
- The mock microbial community contained only 10 species, affecting sensitivity and specificity of taxonomic classification
- The repository README provides no runnable code, and the residual stress is only plotted in supplementary material S2

## Evidence

- [results] Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites: "Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [abstract] significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses: "significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses"
- [results] The residual stress is plotted in supplementary material S2: "The residual stress is plotted in supplementary material S2"
