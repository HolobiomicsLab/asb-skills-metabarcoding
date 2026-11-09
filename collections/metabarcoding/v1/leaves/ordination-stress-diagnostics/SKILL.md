---
name: ordination-stress-diagnostics
description: 'Use when genus-level ASV abundance tables must be compared across sample
  groups (e.g., 16S hypervariable regions) by beta diversity: build a Bray–Curtis
  dissimilarity matrix, ordinate with NMDS, and diagnose the solution''s stress and
  group separation/overlap.'
when_to_use_negative:
- Input is raw FASTQ reads or uncollapsed ASV tables — taxonomy must first be assigned
  with Greengenes and collapsed at the genus level before Bray–Curtis ordination.
- Input tables come from different hypervariable regions than V1–V2/V3–V4/V5–V7/V7–V9
  or from non-16S data where the region-group overlap interpretation does not apply.
- The goal is alpha-diversity comparison (Kruskal–Wallis test on richness/diversity
  indices) rather than between-sample compositional dissimilarity.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: vegan
  role: Computes the Bray–Curtis dissimilarity matrix and runs the metaMDS NMDS ordination;
    stressplot() provides the residual-stress diagnostic
- name: phyloseq
  role: Handles the genus-level collapsed ASV abundance table and sample metadata
    used to build the count matrix for ordination
- name: ggplot2
  role: Draws the NMDS ordination plot with region-group color coding and 0.95 confidence
    ellipses
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:21200c1f5296d17b8a1383b5c83ac40913ac0ce0dd4d4ffa93b88dbd45ffed93
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - phyloseq
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ordination-stress-diagnostics

## Summary

Reproduces NMDS ordination of Bray–Curtis dissimilarities over genus-level 16S ASV counts (Greengenes-collapsed) to visualize which hypervariable region combinations (V1–V2, V3–V4, V5–V7, V7–V9) overlap in ordination space, with stress and permutational fit statistics (R2 = 0.44, pAdonis < 0.001) as diagnostics.

## When to use

Use when genus-level ASV abundance tables must be compared across sample groups (e.g., 16S hypervariable regions) by beta diversity: build a Bray–Curtis dissimilarity matrix, ordinate with NMDS, and diagnose the solution's stress and group separation/overlap.

## When NOT to use

- Input is raw FASTQ reads or uncollapsed ASV tables — taxonomy must first be assigned with Greengenes and collapsed at the genus level before Bray–Curtis ordination.
- Input tables come from different hypervariable regions than V1–V2/V3–V4/V5–V7/V7–V9 or from non-16S data where the region-group overlap interpretation does not apply.
- The goal is alpha-diversity comparison (Kruskal–Wallis test on richness/diversity indices) rather than between-sample compositional dissimilarity.

## Inputs

- Genus-level collapsed ASV abundance table (Greengenes-assigned, produced upstream in QIIME 2)
- Sample-to-hypervariable-region grouping labels (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- NMDS ordination figure with samples color-coded by region group and 0.95 confidence ellipses
- NMDS stress plot (residual stress diagnostic, mirroring supplementary S2)
- Final NMDS stress value (between 0 and 1)
- Fit statistics for the region grouping (R2 = 0.44, pAdonis < 0.001)

## How to apply

Load the genus-level collapsed ASV abundance table produced upstream in QIIME 2 (Greengenes assignments collapsed at genus level) together with sample-to-region grouping labels for V1–V2, V3–V4, V5–V7, and V7–V9. Compute the Bray–Curtis dissimilarity matrix between samples on the genus-level counts and run vegan::metaMDS on that matrix, recording the final stress value (reported as a value between 0 and 1). Plot the NMDS solution with ggplot2, color-coding samples by hypervariable region group and drawing 0.95 confidence ellipses, mirroring Figure 5 of the paper. Generate a stressplot() diagnostic, mirroring the residual-stress plot in supplementary material S2. Report fit statistics from permutational testing of the region grouping (R2 = 0.44, pAdonis < 0.001) and validate that the V3–V4 and V5–V7 groups overlap in both NMDS1 and NMDS2 while V1–V2 and V7–V9 separate.

## Related tools

- **vegan** (Computes the Bray–Curtis dissimilarity matrix and runs the metaMDS NMDS ordination; stressplot() provides the residual-stress diagnostic)
- **phyloseq** (Handles the genus-level collapsed ASV abundance table and sample metadata used to build the count matrix for ordination)
- **ggplot2** (Draws the NMDS ordination plot with region-group color coding and 0.95 confidence ellipses)

## Examples

```
ord <- vegan::metaMDS(t(genus_counts), distance = "bray"); stressplot(ord); ggplot(ord$points) + stat_ellipse(level = 0.95)
```

## Evaluation signals

- The final NMDS stress value is reported and lies between 0 and 1.
- The ordination figure shows the V3–V4 and V5–V7 groups overlapping in both NMDS1 and NMDS2, while V1–V2 and V7–V9 separate from them.
- Permutational fit statistics for the region grouping reproduce the reported R2 = 0.44 with pAdonis < 0.001.
- A residual-stress plot (analogous to supplementary material S2) is generated as a diagnostic of the NMDS solution.
- The output ordination figure file exists and is produced from the genus-level collapsed table, not raw counts.

## Limitations

- The residual stress plot was only published in supplementary material S2, so full stress diagnostics must be regenerated from the ordination rather than read from the main figure.
- Regions were trained with the Greengenes database despite more updated SILVA versions being available, which may bias genus-level assignments feeding the dissimilarity matrix.
- The 300 bp read length limit of the Illumina MiSeq platform constrains 16S rRNA profiling, affecting the underlying ASV counts being ordinated.
- No code or data availability statement was found in the discussion, so exact inputs for reproduction may need to be requested from the authors.

## Evidence

- [results] Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites: "Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [abstract] significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses: "significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses"
- [supplementary] The residual stress is plotted in supplementary material S2: "The residual stress is plotted in supplementary material S2"
