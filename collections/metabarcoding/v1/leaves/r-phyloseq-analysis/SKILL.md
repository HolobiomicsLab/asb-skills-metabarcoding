---
name: r-phyloseq-analysis
description: Use when a genus-level ASV abundance table must be grouped by experimental
  condition (e.g., 16S hypervariable region amplified) and compared for alpha diversity
  (Shannon, inverse Simpson, Chao1) with non-parametric testing in R.
when_to_use_negative:
- Input is already summarized diversity values or a taxa-composition table without
  per-sample ASV counts — richness indices like Chao1 cannot be recomputed from collapsed
  summaries.
- Data are full-length 16S rRNA or shotgun metagenomic reads with no hypervariable-region
  grouping variable, since the comparison framework is specifically across region
  combinations.
- The goal is between-sample beta diversity (compositional dissimilarity, e.g., NMDS
  of Bray–Curtis), which this paper analyzed separately rather than via Kruskal–Wallis
  on alpha indices.
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3697
tools:
- name: phyloseq
  role: R package used to build the ASV-by-sample object and estimate Chao1, Shannon,
    and inverse Simpson diversity indices per sample
- name: vegan
  role: R package used alongside phyloseq for alpha diversity index estimation and
    rarefaction curves in R 3.4.1
- name: Deblur
  role: Upstream denoising producing the genus-level ASV table that serves as the
    count input for phyloseq diversity estimation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:6f832a33bde78ff4d0eecdd41cb6d288891050200532fa7544e2c78269aa1a03
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - R
  - phyloseq
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# r-phyloseq-analysis

## Summary

Estimates and statistically compares alpha diversity indices (Shannon, inverse Simpson, Chao1) across sample groups — here 16S rRNA hypervariable region combinations (V1–V2, V3–V4, V5–V7, V7–V9) — using the phyloseq and vegan packages in R, followed by Kruskal–Wallis testing.

## When to use

Use when a genus-level ASV abundance table must be grouped by experimental condition (e.g., 16S hypervariable region amplified) and compared for alpha diversity (Shannon, inverse Simpson, Chao1) with non-parametric testing in R.

## When NOT to use

- Input is already summarized diversity values or a taxa-composition table without per-sample ASV counts — richness indices like Chao1 cannot be recomputed from collapsed summaries.
- Data are full-length 16S rRNA or shotgun metagenomic reads with no hypervariable-region grouping variable, since the comparison framework is specifically across region combinations.
- The goal is between-sample beta diversity (compositional dissimilarity, e.g., NMDS of Bray–Curtis), which this paper analyzed separately rather than via Kruskal–Wallis on alpha indices.

## Inputs

- Genus-level ASV abundance table (Deblur-denoised, Greengenes-classified, collapsed at genus level)
- Sample metadata mapping each of the 33 sputum samples to its amplified hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)
- phyloseq sample_data grouping variable (e.g., region column)

## Outputs

- Per-sample alpha diversity table with Shannon, inverse Simpson, and Chao1 indices
- Rarefaction curves per sample/region group
- Kruskal–Wallis p-values per diversity index (e.g., Shannon p = 4.6E-07, invSimpson p = 1E-07, Chao1 p = 7.1E-06)
- Ranked list of hypervariable regions from highest to lowest alpha diversity

## How to apply

Load the genus-level ASV abundance table (from Deblur-denoised reads taxonomically assigned with the Greengenes database and collapsed at the genus level) plus sample metadata recording which hypervariable region was amplified for each sputum sample. In R (version 3.4.1), build a phyloseq object and estimate per-sample Chao1 (richness), Shannon index (evenness/entropy), and inverse Simpson index (dominance), optionally alongside rarefaction curves to check sequencing depth adequacy. Run a Kruskal–Wallis test for each of the three indices to detect significant differences among the four region groups (V1–V2, V3–V4, V5–V7, V7–V9). Compile the resulting p-values into a results table and rank the regions from highest to lowest alpha diversity, verifying that V7–V9 ranks lowest. The rationale is that Deblur ASVs collapsed at genus level are the recommended input unit for phyloseq richness/entropy estimation, and the Kruskal–Wallis test handles the non-normal distribution of diversity indices across groups.

## Related tools

- **phyloseq** (R package used to build the ASV-by-sample object and estimate Chao1, Shannon, and inverse Simpson diversity indices per sample)
- **vegan** (R package used alongside phyloseq for alpha diversity index estimation and rarefaction curves in R 3.4.1)
- **Deblur** (Upstream denoising producing the genus-level ASV table that serves as the count input for phyloseq diversity estimation)

## Examples

```
library(phyloseq); ad <- estimate_richness(ps, measures = c("Shannon", "invSimpson", "Chao1")); kruskal.test(ad$Shannon, sample_data(ps)$region)
```

## Evaluation signals

- Reproduced Kruskal–Wallis p-values match article-reported values within numerical tolerance: pShannon = 4.6E-07, pinvSimpson = 1E-07, pChao1 = 7.1E-06.
- V7–V9 ranks lowest of the four regions for all three indices (Shannon, inverse Simpson, Chao1).
- V1–V2, V3–V4, and V5–V7 each show significantly higher alpha diversity than V7–V9.
- Rarefaction curves are generated and indicate adequate sampling depth before interpreting richness differences.
- Output table contains one diversity value per sample (33 sputum samples) for each of the three indices, grouped by the four region combinations.

## Limitations

- The Illumina MiSeq 300 bp length limit constrains 16S rRNA profiling of longer region combinations such as V5–V7 and V7–V9.
- Taxonomy was assigned with Greengenes despite more updated SILVA versions being available, which may affect genus-level assignments feeding the diversity estimates.
- Richness and entropy are niche dependent, so results from respiratory sputum samples lack direct comparability with studies of other environments.
- The ZymoBIOMICS mock community contains only 10 species, affecting sensitivity and specificity of taxonomic classification used to validate the pipeline.

## Evidence

- [abstract] Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9: "Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9"
- [methods] The alpha diversity indices and rarefaction curves were estimated in R (version 3.4.1): "The alpha diversity indices and rarefaction curves were estimated in R (version 3.4.1)"
- [results] to observe significant differences between groups based on the Kruskal–Wallis test: "to observe significant differences between groups based on the Kruskal–Wallis test"
- [methods] Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
