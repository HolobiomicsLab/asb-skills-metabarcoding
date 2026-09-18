---
name: alpha-diversity-index-calculation
description: Use when comparing within-sample diversity (richness/evenness) across
  categorical groups — e.g., 16S hypervariable regions V1–V2, V3–V4, V5–V7, V7–V9
  — on a genus-level ASV count table.
when_to_use_negative:
- When the input is already a diversity summary table rather than a per-sample ASV/genus
  abundance matrix (nothing to estimate; proceed directly to statistical testing).
- When the analysis goal is between-sample compositional dissimilarity (beta diversity,
  e.g., NMDS of Bray–Curtis dissimilarities) rather than within-sample richness/evenness.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3316
tools:
- name: phyloseq
  role: R package used to compute per-sample Chao1, Shannon, and inverse Simpson alpha
    diversity indices on the genus-level ASV table
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- name: vegan
  role: R package used alongside phyloseq for alpha diversity estimation (version
    R 3.4.1)
  repo: github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-023-30764-z
    title: Determining the most accurate 16S rRNA hypervariable region for taxonomic
      identification from respiratory samples
  source_content_hash: sha256:6be810262adae92cfa1963cfb829d892f8c491523c999491493308037a4388fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
  tools:
  - R
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# alpha-diversity-index-calculation

## Summary

Estimate per-sample alpha diversity (Chao1 richness, Shannon entropy, and inverse Simpson dominance) on genus-level 16S rRNA ASV abundance tables in R using phyloseq and vegan, then test for differences between sample groups (e.g., hypervariable regions) with the Kruskal–Wallis test.

## When to use

Use when comparing within-sample diversity (richness/evenness) across categorical groups — e.g., 16S hypervariable regions V1–V2, V3–V4, V5–V7, V7–V9 — on a genus-level ASV count table.

## When NOT to use

- When the input is already a diversity summary table rather than a per-sample ASV/genus abundance matrix (nothing to estimate; proceed directly to statistical testing).
- When the analysis goal is between-sample compositional dissimilarity (beta diversity, e.g., NMDS of Bray–Curtis dissimilarities) rather than within-sample richness/evenness.

## Inputs

- Genus-level ASV abundance table from the study GitHub repository (github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification)
- Sample metadata mapping each sputum sample to its amplified hypervariable region (V1–V2, V3–V4, V5–V7, V7–V9)

## Outputs

- Per-sample alpha diversity estimates for Chao1, Shannon, and inverse Simpson indices
- Kruskal–Wallis p-values per index (expected: pShannon = 4.6E-07, pinvSimpson = 1E-07, pChao1 = 7.1E-06)
- Ranking of hypervariable regions from highest to lowest alpha diversity

## How to apply

Load the genus-level ASV abundance table from the study repository and group samples by the hypervariable region amplified (V1–V2, V3–V4, V5–V7, V7–V9). Estimate alpha diversity per sample in R (version 3.4.1) with the phyloseq and vegan packages, computing Chao1 (richness), Shannon index (evenness/entropy), and inverse Simpson index (dominance). Run a separate Kruskal–Wallis test for each of the three indices to test for significant differences among the four region groups. Compile the p-values into a results table and rank regions from highest to lowest alpha diversity, verifying that V7–V9 ranks lowest. Validate by confirming the reproduced p-values match the article-reported values (pShannon = 4.6E-07, pinvSimpson = 1E-07, pChao1 = 7.1E-06) within numerical tolerance and that V1–V2, V3–V4, and V5–V7 each show significantly higher alpha diversity than V7–V9.

## Related tools

- **phyloseq** (R package used to compute per-sample Chao1, Shannon, and inverse Simpson alpha diversity indices on the genus-level ASV table) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification
- **vegan** (R package used alongside phyloseq for alpha diversity estimation (version R 3.4.1)) — github.com/ruben-aladid/-Hypervariable-regions-of-16S-rRNA-with-the-highest-resolving-power-for-taxonomic-identification

## Examples

```
library(phyloseq); div <- estimate_richness(ps, measures = c("Shannon", "invSimpson", "Chao1")); kruskal.test(div$Shannon ~ sample_data(ps)$region)
```

## Evaluation signals

- Reproduced Kruskal–Wallis p-values match the article-reported values within numerical tolerance: Shannon p = 4.6E-07, inverse Simpson p = 1E-07, Chao1 p = 7.1E-06.
- V7–V9 ranks lowest for all three indices (Shannon, inverse Simpson, Chao1) compared with V1–V2, V3–V4, and V5–V7.
- Each of V1–V2, V3–V4, and V5–V7 shows significantly higher alpha diversity than V7–V9 in post-hoc comparison.
- Indices are computed on the genus-level ASV table collapsed via the Greengenes database, matching the article's taxonomy assignment step.
- R session uses version 3.4.1 with phyloseq and vegan, as in the original workflow.

## Limitations

- Richness and entropy are niche dependent, so results from these sputum samples are not directly comparable to studies in other environments or body sites.
- Taxonomy was assigned with Greengenes rather than more updated versions of SILVA, which may bias genus-level counts feeding into the diversity estimates.
- The Illumina MiSeq 300 bp read-length limit constrains 16S rRNA profiling, affecting which region combinations can be sequenced and compared.
- The rarefaction curves accompanying the diversity estimates were generated in R (version 3.4.1); deviations from that pipeline may alter Chao1-richness estimates.

## Evidence

- [abstract] Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9: "Alpha diversity was significantly higher for V1–V2, V3–V4, and V5–V7 compared with V7–V9"
- [methods] Alpha diversity indices estimated in R: "The alpha diversity indices and rarefaction curves were estimated in R (version 3.4.1)"
- [results] Kruskal–Wallis test between groups: "to observe significant differences between groups based on the Kruskal–Wallis test"
- [methods] Taxonomic assignments collapsed at genus level: "Taxonomic assignments were obtained using the Greengenes database and collapsed at the genus level"
- [discussion] Richness or entropy are niche dependent: "Most of the other studies discussed here lack direct comparability because richness or entropy vary depending on the actual composition, which is niche dependent"
