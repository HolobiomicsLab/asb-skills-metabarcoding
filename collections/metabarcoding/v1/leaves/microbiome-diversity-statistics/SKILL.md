---
name: microbiome-diversity-statistics
description: Use when a genus-level ASV abundance table is grouped by experimental
  condition or amplified region and you must test whether alpha diversity (richness/evenness)
  differs significantly across groups.
when_to_use_negative:
- The input is raw FASTQ reads or an uncollapsed ASV table — this skill requires a
  genus-level, Greengenes-collapsed abundance table after Deblur denoising
- The analysis goal is beta diversity / compositional dissimilarity (e.g., NMDS of
  Bray–Curtis dissimilarities), which is a separate analysis in this study
- Groups are expected to be normally distributed and homoscedastic, where a parametric
  ANOVA rather than Kruskal–Wallis would be appropriate
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3053
tools:
- name: phyloseq
  role: R package used to compute alpha diversity indices (Chao1, Shannon, inverse
    Simpson) from the genus-level ASV table
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: vegan
  role: R package used alongside phyloseq to estimate alpha diversity indices and
    rarefaction curves
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: Deblur
  role: Upstream denoising step that produces the bacterial ASVs at the genus level
    feeding the diversity calculations
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:89764a1dbfda0d6cd48548d28ae7e1a5adea8cc0b238874bea3cd28c846d7bb9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - Deblur
  - R
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# microbiome-diversity-statistics

## Summary

Compares alpha diversity (Shannon, inverse Simpson, Chao1) across 16S rRNA hypervariable-region groups (V1–V2, V3–V4, V5–V7, V7–V9) using the Kruskal–Wallis test in R, to determine which region combination yields the highest richness and evenness in respiratory samples.

## When to use

Use when a genus-level ASV abundance table is grouped by experimental condition or amplified region and you must test whether alpha diversity (richness/evenness) differs significantly across groups.

## When NOT to use

- The input is raw FASTQ reads or an uncollapsed ASV table — this skill requires a genus-level, Greengenes-collapsed abundance table after Deblur denoising
- The analysis goal is beta diversity / compositional dissimilarity (e.g., NMDS of Bray–Curtis dissimilarities), which is a separate analysis in this study
- Groups are expected to be normally distributed and homoscedastic, where a parametric ANOVA rather than Kruskal–Wallis would be appropriate

## Inputs

- Genus-level ASV abundance table (Greengenes-assigned, collapsed at genus level) from 33 sputum samples
- Sample metadata mapping each sample to its amplified region group (V1–V2, V3–V4, V5–V7, V7–V9)
- phyloseq/vegan-compatible phyloseq object in R

## Outputs

- Per-sample alpha diversity values for Shannon, inverse Simpson, and Chao1 indices
- Rarefaction curves per region group
- Kruskal–Wallis p-values per index (expected: pShannon = 4.6E-07, pinvSimpson = 1E-07, pChao1 = 7.1E-06)
- Ranked ordering of regions by alpha diversity confirming V7–V9 lowest

## How to apply

Load the genus-level ASV abundance table from the study repository (taxonomic assignments collapsed at the genus level via Greengenes) and group sputum samples by amplified hypervariable region. In R (version 3.4.1), estimate Chao1 (richness), Shannon index (evenness/entropy), and inverse Simpson index (dominance) per sample using the phyloseq and vegan packages, alongside rarefaction curves to check sequencing depth adequacy. Run a Kruskal–Wallis test per index to test for differences among the four region groups, since diversity values are non-normally distributed across samples. Compile p-values into a results table, rank regions highest-to-lowest, and verify that V7–V9 ranks lowest. Validate by reproducing the reported statistics: Shannon p = 4.6E-07, inverse Simpson p = 1E-07, Chao1 p = 7.1E-06, with V1–V2, V3–V4, and V5–V7 each significantly higher than V7–V9.

## Related tools

- **phyloseq** (R package used to compute alpha diversity indices (Chao1, Shannon, inverse Simpson) from the genus-level ASV table) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **vegan** (R package used alongside phyloseq to estimate alpha diversity indices and rarefaction curves) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **Deblur** (Upstream denoising step that produces the bacterial ASVs at the genus level feeding the diversity calculations)

## Examples

```
library(phyloseq); div <- estimate_richness(physeq, measures=c("Shannon","invsimpson","Chao1")); kruskal.test(div$Shannon ~ sample_data(physeq)$region)
```

## Evaluation signals

- Reproduced Kruskal–Wallis p-values match the article within numerical tolerance: Shannon p = 4.6E-07, inverse Simpson p = 1E-07, Chao1 p = 7.1E-06
- All three indices are significantly decreased in the V7–V9 group compared with the other hypervariable regions
- Region ranking from highest to lowest alpha diversity places V1–V2, V3–V4, and V5–V7 above V7–V9
- Rarefaction curves are generated alongside diversity estimates to confirm indices were computed at comparable sequencing depth (Q30-filtered reads)
- Output table contains one Kruskal–Wallis p-value per diversity index (exactly three tests for three indices)

## Limitations

- Rarefaction/diversity results are niche-dependent: richness or entropy varies with actual sample composition, limiting cross-study comparability
- The Illumina MiSeq 300 bp length limit constrains which region combinations (e.g., V7–V9) can be profiled, potentially biasing diversity estimates for that group
- Taxonomic assignments relied on Greengenes rather than updated SILVA versions, which may affect the genus-level table underlying the diversity computation

## Evidence

- [abstract] Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9: "Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9"
- [methods] Alpha diversity indices estimated in R 3.4.1: "The alpha diversity indices and rarefaction curves were estimated in R (version 3.4.1)"
- [results] Kruskal–Wallis test for group differences: "to observe significant differences between groups based on the Kruskal–Wallis test"
- [methods] Taxonomic collapsing at genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
