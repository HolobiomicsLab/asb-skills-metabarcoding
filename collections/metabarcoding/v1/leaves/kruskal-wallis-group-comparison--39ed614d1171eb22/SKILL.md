---
name: kruskal-wallis-group-comparison
description: Use when per-sample alpha diversity indices (Shannon, inverse Simpson,
  Chao1) computed from a genus-level ASV abundance table must be compared across discrete
  groups, such as samples grouped by the 16S rRNA hypervariable region amplified,
  to test whether one region has significantly lower diversity.
when_to_use_negative:
- The analysis goal is beta diversity or compositional dissimilarity (e.g. NMDS ordination
  of Bray–Curtis dissimilarities between sites), not per-sample alpha diversity group
  comparison.
- Groups are not discrete categories (e.g. comparing against full-length 16S rRNA
  sequencing or continuous covariates) — the article did not perform a full-length
  16S comparison, and the Kruskal–Wallis design assumes categorical group membership.
- Input is raw reads rather than a collapsed genus-level ASV table — quality filtering
  (FastQC), Deblur denoising, chimera identification, and Greengenes genus-level collapsing
  must happen first.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: R (version 3.4.1)
  role: Environment in which alpha diversity indices are estimated and the Kruskal–Wallis
    test is run
- name: phyloseq
  role: R package used to estimate per-sample alpha diversity indices (Chao1, Shannon,
    inverse Simpson) from the genus-level ASV table
- name: vegan
  role: R package used alongside phyloseq for alpha diversity estimation and rarefaction
    curves
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:152817f0f9420b7275c7f62c2c5ca743b34e2a9263d667ff0800f193d2d1da3b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - R (version 3.4.1)
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# kruskal-wallis-group-comparison

## Summary

A non-parametric Kruskal–Wallis test applied to per-sample alpha diversity indices (Shannon, inverse Simpson, Chao1) to test whether groups defined by amplified 16S rRNA hypervariable regions (V1–V2, V3–V4, V5–V7, V7–V9) differ significantly in diversity, e.g. whether V7–V9 yields lower diversity than other regions.

## When to use

Use when per-sample alpha diversity indices (Shannon, inverse Simpson, Chao1) computed from a genus-level ASV abundance table must be compared across discrete groups, such as samples grouped by the 16S rRNA hypervariable region amplified, to test whether one region has significantly lower diversity.

## When NOT to use

- The analysis goal is beta diversity or compositional dissimilarity (e.g. NMDS ordination of Bray–Curtis dissimilarities between sites), not per-sample alpha diversity group comparison.
- Groups are not discrete categories (e.g. comparing against full-length 16S rRNA sequencing or continuous covariates) — the article did not perform a full-length 16S comparison, and the Kruskal–Wallis design assumes categorical group membership.
- Input is raw reads rather than a collapsed genus-level ASV table — quality filtering (FastQC), Deblur denoising, chimera identification, and Greengenes genus-level collapsing must happen first.

## Inputs

- Genus-level ASV abundance table (from the study repository github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification)
- Sample metadata mapping each sputum sample to its amplified hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)
- Phyloseq/vegan-compatible R object (R version 3.4.1) containing ASV abundances and sample metadata

## Outputs

- Kruskal–Wallis p-value per alpha diversity index (Shannon, inverse Simpson, Chao1)
- Results table ranking hypervariable regions from highest to lowest alpha diversity
- Alpha diversity index values per sample (Shannon, inverse Simpson, Chao1) and supporting rarefaction curves

## How to apply

Load the genus-level ASV abundance table from the study repository and group samples by the hypervariable region amplified (V1–V2, V3–V4, V5–V7, V7–V9). Estimate alpha diversity indices per sample in R (version 3.4.1) with the phyloseq and vegan packages: Chao1 for richness, Shannon for evenness/entropy, and inverse Simpson for dominance. Run a Kruskal–Wallis test for each of the three indices to test for significant differences among the four region groups (article-reported results: pShannon = 4.6E-07, pinvSimpson = 1E-07, pChao1 = 7.1E-06). Compile the resulting p-values into a results table, rank the regions from highest to lowest alpha diversity, and verify that V7–V9 ranks lowest while V1–V2, V3–V4, and V5–V7 each show significantly higher alpha diversity. Rarefaction curves should accompany the indices to confirm sampling depth support.

## Related tools

- **R (version 3.4.1)** (Environment in which alpha diversity indices are estimated and the Kruskal–Wallis test is run)
- **phyloseq** (R package used to estimate per-sample alpha diversity indices (Chao1, Shannon, inverse Simpson) from the genus-level ASV table)
- **vegan** (R package used alongside phyloseq for alpha diversity estimation and rarefaction curves)

## Examples

```
kw_p <- kruskal.test(shannon ~ region, data = alpha_div)$p.value  # alpha_div from phyloseq/vegan indices per sample grouped by hypervariable region
```

## Evaluation signals

- Reproduced p-values match the article-reported values within numerical tolerance: Shannon p = 4.6E-07, Chao1 p = 7.1E-06, inverse Simpson p = 1E-07.
- V7–V9 ranks lowest among the four region groups for all three indices (Shannon, inverse Simpson, Chao1).
- V1–V2, V3–V4, and V5–V7 each show significantly higher alpha diversity than V7–V9.
- Rarefaction curves accompany the alpha diversity estimates, indicating index values are supported by sampling depth.
- All three indices were computed at the genus level after taxonomic assignment with the Greengenes database, consistent with the article's methods.

## Limitations

- Alpha diversity was computed on taxa assigned with the Greengenes database, which the article notes is outdated compared with more updated SILVA versions.
- The 300 bp length limit of the Illumina MiSeq platform limits 16S rRNA profiling and may affect coverage of the longer combined regions.
- Richness or entropy are niche dependent, so results lack direct comparability with other studies.
- Only 33 human sputum samples were analyzed, limiting statistical power for group comparisons.

## Evidence

- [abstract] Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9: "Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9"
- [results] Kruskal–Wallis test for group differences: "to observe significant differences between groups based on the Kruskal–Wallis test"
- [methods] Alpha diversity estimation in R: "The alpha diversity indices and rarefaction curves were estimated in R (version 3.4.1)"
