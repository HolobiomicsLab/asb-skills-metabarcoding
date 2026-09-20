---
name: beta-diversity-analysis-of-16s-microbiome-data
description: Use when a genus-level 16S rRNA abundance table (e.g., Greengenes-collapsed
  ASVs from QIIME 2) exists and the goal is to quantify and visualize compositional
  dissimilarity between sample groups via NMDS ordination of Bray–Curtis dissimilarities.
when_to_use_negative:
- When the input is raw sequencing reads rather than a processed genus-level abundance
  table — upstream steps (FastQC quality filtering with Q30 cutoffs, Deblur denoising,
  chimera identification, Greengenes genus-level collapsing) must run first
- When the research question concerns within-sample richness or alpha diversity (e.g.,
  Kruskal–Wallis comparisons of diversity indices or rarefaction curves) rather than
  between-sample compositional dissimilarity
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3307
tools:
- name: vegan
  role: Computes the Bray–Curtis dissimilarity matrix and runs the metaMDS ordination,
    returning stress and fit statistics
- name: phyloseq
  role: Handles the genus-level ASV abundance table and sample metadata for microbiome
    analysis
- name: ggplot2
  role: Draws the NMDS ordination colored by hypervariable region group with 0.95
    confidence ellipses and the stressplot diagnostic
- name: QIIME 2
  role: Upstream producer of the genus-level collapsed ASV table (Deblur denoising,
    chimera identification, Greengenes taxonomic collapsing) that serves as ordination
    input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:f662de22e311e9a339c34cf3ecd1a11885e70e6052467d5d48afdfc26b32d76b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - ggplot2
  - phyloseq
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# beta-diversity-analysis-of-16s-microbiome-data

## Summary

Non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities on genus-level 16S rRNA ASV tables to test whether sample groups (e.g., hypervariable regions V1–V2, V3–V4, V5–V7, V7–V9) differ compositionally and to visualize overlap or separation in ordination space.

## When to use

Use when a genus-level 16S rRNA abundance table (e.g., Greengenes-collapsed ASVs from QIIME 2) exists and the goal is to quantify and visualize compositional dissimilarity between sample groups via NMDS ordination of Bray–Curtis dissimilarities.

## When NOT to use

- When the input is raw sequencing reads rather than a processed genus-level abundance table — upstream steps (FastQC quality filtering with Q30 cutoffs, Deblur denoising, chimera identification, Greengenes genus-level collapsing) must run first
- When the research question concerns within-sample richness or alpha diversity (e.g., Kruskal–Wallis comparisons of diversity indices or rarefaction curves) rather than between-sample compositional dissimilarity

## Inputs

- Genus-level collapsed ASV abundance table (Greengenes taxonomy, produced in QIIME 2 after Deblur denoising and chimera filtering)
- Sample-to-group metadata mapping samples to 16S hypervariable region combinations (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- NMDS ordination plot (ggplot2) with samples colored by hypervariable region group and 0.95 confidence ellipses
- NMDS stress value (scalar between 0 and 1) and residual stressplot diagnostic
- PERMANOVA fit statistics (R2 and pAdonis)

## How to apply

Load the genus-level collapsed ASV abundance table (Greengenes assignments collapsed at genus level, produced upstream in QIIME 2 via Deblur denoising) together with sample-to-group labels (here, the 16S hypervariable regions V1–V2, V3–V4, V5–V7, V7–V9). Compute the Bray–Curtis dissimilarity matrix between samples and run vegan's metaMDS on it, recording the final stress value (between 0 and 1) and PERMANOVA fit statistics — the paper reports R2 = 0.44 with pAdonis < 0.001. Plot the ordination with ggplot2, color-coding samples by hypervariable region group with 0.95 confidence ellipses, and generate a residual stressplot as a diagnostic (the paper's supplementary material S2 plots residual stress). Validate by confirming the figure reproduces the expected pattern: overlap of the V3–V4 and V5–V7 groups in both NMDS1 and NMDS2, versus separation of V1–V2 and V7–V9.

## Related tools

- **vegan** (Computes the Bray–Curtis dissimilarity matrix and runs the metaMDS ordination, returning stress and fit statistics)
- **phyloseq** (Handles the genus-level ASV abundance table and sample metadata for microbiome analysis)
- **ggplot2** (Draws the NMDS ordination colored by hypervariable region group with 0.95 confidence ellipses and the stressplot diagnostic)
- **QIIME 2** (Upstream producer of the genus-level collapsed ASV table (Deblur denoising, chimera identification, Greengenes taxonomic collapsing) that serves as ordination input)

## Examples

```
R: library(vegan); ord <- metaMDS(t(genus_counts), distance = "bray", k = 2); ord$stress; stressplot(ord)
```

## Evaluation signals

- Stress value from metaMDS is recorded and falls between 0 and 1
- Output ordination figure exists and shows overlap of the V3–V4 and V5–V7 groups in both NMDS1 and NMDS2 dimensions, with V1–V2 and V7–V9 separated
- PERMANOVA fit statistics (R2 and pAdonis) are reported alongside the ordination; reference result is R2 = 0.44, pAdonis < 0.001
- A residual stressplot diagnostic is generated, mirroring the paper's supplementary stressplot S2
- The dissimilarity matrix is computed from genus-level collapsed counts (not raw ASVs or relative abundance of uncollapsed features)

## Limitations

- Reference database mismatch: the different regions were trained with outdated Greengenes instead of more updated SILVA versions, which may affect taxonomic composition feeding the ordination
- The Illumina MiSeq 300 bp length limit constrains 16S rRNA profiling, so region-specific amplification bias can drive apparent compositional differences
- The residual stressplot of the NMDS solution is only in supplementary material S2; the main text does not report the stress value, so comparability of ordination fit must be re-derived
- Findings are niche-dependent: most other studies lack direct comparability because richness or entropy vary depending on actual composition, limiting generalization of the ordination patterns to other niches

## Evidence

- [results] Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites: "Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [results] The residual stress is plotted in supplementary material S2: "The residual stress is plotted in supplementary material S2"
- [abstract] significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses: "significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses"
- [discussion] the different regions were trained with Greengenes: "despite the availability of more updated versions of SILVA, the different regions were trained with Greengenes"
