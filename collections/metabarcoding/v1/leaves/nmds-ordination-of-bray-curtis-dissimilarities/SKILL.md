---
name: nmds-ordination-of-bray-curtis-dissimilarities
description: Use when genus-level ASV abundance tables from different 16S hypervariable-region
  groups need to be compared for compositional dissimilarity via ordination, and when
  group overlap/separation in NMDS space must be visualized and stress-tested.
when_to_use_negative:
- Input is raw FASTQ reads rather than a genus-level collapsed ASV abundance table
  — denoising with Deblur and taxonomic collapsing at the genus level must happen
  first
- The goal is alpha-diversity comparison between region groups (Kruskal–Wallis test)
  rather than between-sample compositional dissimilarity
- Samples belong to a single hypervariable region group, so no between-group ordination
  overlap can be assessed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: vegan
  role: Computes the Bray–Curtis dissimilarity matrix and runs the metaMDS NMDS ordination
    on genus-level counts
- name: phyloseq
  role: Handles the genus-level collapsed ASV abundance table and sample grouping
    data feeding the ordination
- name: ggplot2
  role: Plots the NMDS ordination color-coded by hypervariable region group with 0.95
    confidence ellipses
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:4ac87ac83b7a5041cd64ae48e6edcf4ebcefbf5c57510b74c17eec23d68186ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ggplot2
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# nmds-ordination-of-bray-curtis-dissimilarities

## Summary

Reproduces the paper's non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between sputum 16S rRNA hypervariable-region samples (Figure 5), plus a supplementary stress diagnostic (S2), to reveal which region combinations (V1–V2, V3–V4, V5–V7, V7–V9) overlap or separate in ordination space.

## When to use

Use when genus-level ASV abundance tables from different 16S hypervariable-region groups need to be compared for compositional dissimilarity via ordination, and when group overlap/separation in NMDS space must be visualized and stress-tested.

## When NOT to use

- Input is raw FASTQ reads rather than a genus-level collapsed ASV abundance table — denoising with Deblur and taxonomic collapsing at the genus level must happen first
- The goal is alpha-diversity comparison between region groups (Kruskal–Wallis test) rather than between-sample compositional dissimilarity
- Samples belong to a single hypervariable region group, so no between-group ordination overlap can be assessed

## Inputs

- Genus-level collapsed ASV abundance table (Greengenes assignments, QIIME 2 output)
- Sample-to-hypervariable-region grouping labels (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- NMDS ordination figure (NMDS1 vs NMDS2, color-coded by region group with 0.95 confidence ellipses)
- NMDS stress value and fit statistics (R2, pAdonis)
- Supplementary-style residual stress plot (stress diagnostic)

## How to apply

Load the genus-level collapsed ASV abundance table (Greengenes assignments collapsed at genus level, produced upstream in QIIME 2) together with the sample-to-region grouping labels for V1–V2, V3–V4, V5–V7 and V7–V9. Compute the Bray–Curtis dissimilarity matrix between samples with the vegan metaMDS workflow, run NMDS on the dissimilarity matrix, and record the final stress value and fit statistics (the paper reports R2 = 0.44, pAdonis < 0.001). Plot the ordination with ggplot2, color-coding samples by hypervariable region group and drawing 0.95 confidence ellipses. Additionally generate a residual stress plot mirroring supplementary material S2 as a diagnostic. Validate that V3–V4 and V5–V7 overlap in both NMDS1 and NMDS2 dimensions while V1–V2 and V7–V9 remain separated.

## Related tools

- **vegan** (Computes the Bray–Curtis dissimilarity matrix and runs the metaMDS NMDS ordination on genus-level counts)
- **phyloseq** (Handles the genus-level collapsed ASV abundance table and sample grouping data feeding the ordination)
- **ggplot2** (Plots the NMDS ordination color-coded by hypervariable region group with 0.95 confidence ellipses)

## Examples

```
Rscript -e 'library(vegan); ord <- metaMDS(genus_counts, distance = "bray"); ord$stress; plot(ord)'
```

## Evaluation signals

- Output figure exists and reproduces the reported pattern: V3–V4 and V5–V7 overlap in both NMDS1 and NMDS2 dimensions, while V1–V2 and V7–V9 separate
- Stress value recorded and falls between 0 and 1
- Fit statistics reported, matching the paper's R2 = 0.44 and pAdonis < 0.001 (adonis on the Bray–Curtis matrix against region labels)
- A residual stress plot mirroring supplementary material S2 is generated as a diagnostic
- Ordination is based on a Bray–Curtis dissimilarity matrix computed on genus-level collapsed counts (not raw ASVs or other distances)

## Limitations

- The residual stress of the NMDS solution is only available in supplementary material S2; the main text does not report it directly
- No code or data availability statement accompanies the discussion, limiting reproducibility beyond the GitHub repository
- Taxonomic assignments rely on the outdated Greengenes database instead of updated SILVA versions, which can bias genus-level dissimilarities
- The 300 bp length limit of the Illumina MiSeq platform constrains 16S rRNA profiling and hence the compositional profiles being ordinated

## Evidence

- [results] Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites: "Figure 5 shows the results for non-metric multidimensional scaling (NMDS) ordination of Bray–Curtis dissimilarities between different sites"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [abstract] significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses: "significant compositional dissimilarities in the V1–V2 and V7–V9 analyses versus the V3–V4 and V5–V7 analyses"
- [supplementary] The residual stress is plotted in supplementary material S2: "The residual stress is plotted in supplementary material S2"
