---
name: r-phyloseq-vegan-microbiome-workflow
description: Use when genus-level ASV abundance tables derived from 16S rRNA hypervariable
  region comparisons need ordination-based compositional dissimilarity assessment
  and group-overlap visualization in NMDS space.
when_to_use_negative:
- Input is raw reads or uncollapsed ASV table — taxonomic collapsing at genus level
  (Greengenes) must occur upstream first
- The goal is alpha diversity or richness testing (Kruskal–Wallis, rarefaction curves)
  rather than between-sample compositional ordination
- Data come from full-length 16S rRNA sequencing or non-16S amplicon data where hypervariable-region
  grouping labels are not defined
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: vegan
  role: Computes Bray–Curtis dissimilarity matrix and runs NMDS ordination via metaMDS;
    provides stress statistics
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: phyloseq
  role: Handles genus-level collapsed ASV abundance table and sample metadata for
    microbiome-specific data structures
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: ggplot2
  role: Renders the NMDS ordination scatter plot with region-colored points and 0.95
    confidence ellipses
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:33801b8db2faa97aec709952ab325d2c2aae6d9316f74c1ee693dcdc6b129a8e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - ggplot2
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# r-phyloseq-vegan-microbiome-workflow

## Summary

An R-based microbiome ordination workflow that computes Bray–Curtis dissimilarities on genus-level 16S rRNA amplicon data and visualizes sample clustering by hypervariable region using NMDS, to test whether region combinations yield distinct or overlapping compositional profiles.

## When to use

Use when genus-level ASV abundance tables derived from 16S rRNA hypervariable region comparisons need ordination-based compositional dissimilarity assessment and group-overlap visualization in NMDS space.

## When NOT to use

- Input is raw reads or uncollapsed ASV table — taxonomic collapsing at genus level (Greengenes) must occur upstream first
- The goal is alpha diversity or richness testing (Kruskal–Wallis, rarefaction curves) rather than between-sample compositional ordination
- Data come from full-length 16S rRNA sequencing or non-16S amplicon data where hypervariable-region grouping labels are not defined

## Inputs

- Genus-level collapsed ASV abundance table (Greengenes assignments, collapsed at genus level, QIIME 2 output)
- Sample-to-hypervariable-region grouping labels (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- NMDS ordination plot colored by hypervariable region group with 0.95 confidence ellipses
- Stress plot (scree/diagnostic) of the NMDS solution
- Final NMDS stress value and fit statistics (R2, pAdonis)
- Bray–Curtis dissimilarity matrix between samples

## How to apply

Load the genus-level collapsed ASV abundance table (Greengenes assignments collapsed at genus level, produced upstream in QIIME 2) together with sample-to-region grouping labels (V1–V2, V3–V4, V5–V7, V7–V9). Compute the Bray–Curtis dissimilarity matrix between samples using the vegan package metaMDS workflow on the genus-level counts, then run NMDS ordination with metaMDS and record the final stress value and fit statistics (reported R2 = 0.44, pAdonis < 0.001). Plot the ordination with ggplot2, color-coding samples by hypervariable region group and drawing 0.95 confidence ellipses. Generate a stress plot of the NMDS solution as a diagnostic (mirroring supplementary material S2). Validate that the figure reproduces the reported pattern: overlap of V3–V4 and V5–V7 groups in both NMDS1 and NMDS2 versus separation of V1–V2 and V7–V9, with stress recorded between 0 and 1.

## Related tools

- **vegan** (Computes Bray–Curtis dissimilarity matrix and runs NMDS ordination via metaMDS; provides stress statistics) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **phyloseq** (Handles genus-level collapsed ASV abundance table and sample metadata for microbiome-specific data structures) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **ggplot2** (Renders the NMDS ordination scatter plot with region-colored points and 0.95 confidence ellipses) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification

## Examples

```
library(vegan); ord <- metaMDS(t(genus_counts), distance = "bray", k = 2); ordiplot(ord, type = "n"); ordiellipse(ord, region_labels, kind = "ehull", conf = 0.95)
```

## Evaluation signals

- Output figure exists and shows V3–V4 and V5–V7 groups overlapping in both NMDS1 and NMDS2 dimensions
- V1–V2 and V7–V9 groups are visibly separated from the V3–V4/V5–V7 cluster in ordination space
- Recorded NMDS stress value falls between 0 and 1
- Fit statistics reported include R2 = 0.44 and pAdonis < 0.001
- A supplementary-style stress plot is generated for diagnostic verification

## Limitations

- Residual stress of the NMDS is only plotted in supplementary material S2; main text does not report it directly
- Regions were trained with Greengenes instead of more updated SILVA versions, which may bias taxonomic assignment feeding the ordination
- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling and thus region-resolved compositional comparisons
- No comparison to full-length 16S rRNA sequencing was performed to validate the ordination-based grouping

## Evidence

- [results] NMDS ordination of Bray–Curtis dissimilarities between sites: "Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites"
- [abstract] Significant compositional dissimilarities across region combinations: "significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses"
- [methods] Taxonomic assignments collapsed at genus level with Greengenes: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [results] Residual stress plotted in supplementary material S2: "The residual stress is plotted in supplementary material S2"
- [discussion] MiSeq 300 bp limit constrains 16S profiling: "the 300 bp length limit of the Illumna MiSeq platform limits 16S rRNA profiling"
