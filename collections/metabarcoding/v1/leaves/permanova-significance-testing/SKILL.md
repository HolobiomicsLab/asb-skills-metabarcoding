---
name: permanova-significance-testing
description: Use when you have an amplicon feature table (BIOM/QIIME 2 format) plus
  a sample metadata mapping file and need a p-value for whether defined groups separate
  in beta-diversity space.
when_to_use_negative:
- The question is about alpha-diversity differences (within-sample richness/diversity
  indices) — use ANOVA, Mann-Whitney U, or Kruskal-Wallis with boxplots instead
- Input is raw sequencing reads rather than a processed feature table with a metadata
  mapping file
- There are no predefined sample groups to compare — PERMANOVA tests group separation
  and is meaningless on ungrouped samples
edam_operation: http://edamontology.org/operation_2263
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3697
tools:
- name: vegan
  role: R package providing adonis for PERMANOVA on the distance matrix and PCoA/NMDS/CPCoA
    ordination of samples
- name: QIIME 2
  role: Produces the amplicon feature table (BIOM/QIIME 2 format) and distance matrices
    that PERMANOVA is applied to
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:5fada09ddf478a81611e5b505dcd93a8e0036542d2fec243b4629fbad68e1920
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - vegan
  edam_operation: http://edamontology.org/operation_2263
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# permanova-significance-testing

## Summary

Assess whether experimental groups differ significantly in beta-diversity by running PERMANOVA (vegan::adonis) on a sample-sample distance matrix (e.g. Bray-Curtis, UniFrac, Jaccard), complementing ordination scatter plots used to visualize group separation.

## When to use

Use when you have an amplicon feature table (BIOM/QIIME 2 format) plus a sample metadata mapping file and need a p-value for whether defined groups separate in beta-diversity space.

## When NOT to use

- The question is about alpha-diversity differences (within-sample richness/diversity indices) — use ANOVA, Mann-Whitney U, or Kruskal-Wallis with boxplots instead
- Input is raw sequencing reads rather than a processed feature table with a metadata mapping file
- There are no predefined sample groups to compare — PERMANOVA tests group separation and is meaningless on ungrouped samples

## Inputs

- Amplicon feature table (BIOM or QIIME 2 format)
- Sample metadata mapping file with group assignments
- Beta-diversity distance matrix (Bray-Curtis, UniFrac, or Jaccard) computed between samples

## Outputs

- PERMANOVA test statistic and p-value for group separation
- Ordination scatter plot (PCoA/NMDS/CPCoA) colored by group
- Group-difference comparison table and figure consistent with metadata group assignments

## How to apply

Load the BIOM/QIIME 2 feature table and metadata mapping file into R, then compute a beta-diversity distance matrix between samples choosing Bray-Curtis, UniFrac, or Jaccard depending on whether abundance and phylogeny should be weighted. Ordinate the matrix first (PCoA, NMDS, or constrained CPCoA from the vegan package) to check whether groups visually separate before formal testing. Run PERMANOVA with vegan's adonis on the distance matrix using the grouping column from the metadata file as the model formula, so the permutation test evaluates between-group vs within-group dispersion. Visualize the same matrix as a PCoA/NMDS scatter plot with ggplot2 and assemble the PERMANOVA statistic, p-value, and ordination figure into the group-difference comparison table. The rationale, per Liu et al. 2020, is that beta-diversity differences between groups are assessed with PERMANOVA together with ordination scatter plots, while alpha-diversity differences use ANOVA, Mann-Whitney U, or Kruskal-Wallis tests with boxplots — a different test family that should not be substituted for PERMANOVA.

## Related tools

- **vegan** (R package providing adonis for PERMANOVA on the distance matrix and PCoA/NMDS/CPCoA ordination of samples)
- **QIIME 2** (Produces the amplicon feature table (BIOM/QIIME 2 format) and distance matrices that PERMANOVA is applied to)

## Examples

```
adonis(bray_curtis_dist ~ group, data = metadata, permutations = 999)  # vegan::adonis PERMANOVA on Bray-Curtis distance matrix
```

## Evaluation signals

- adonis returns a non-negative F-like statistic with a p-value computed by permutations, and the permutation p-value must fall within [0,1]
- The model formula grouping variable must match sample IDs in the metadata mapping file — mismatved sample names indicate an input error
- PERMANOVA result should be directionally consistent with the PCoA/NMDS ordination scatter plot (significant p-value accompanied by visible group separation)
- Validation: confirm that the alpha/beta test statistics and p-values, all figure files, and the difference-comparison table are generated and consistent with the metadata group assignments
- Changing the distance matrix (Bray-Curtis vs Jaccard vs UniFrac) may legitimately change the result; identical results across all metrics should prompt a check that the correct matrix was loaded

## Limitations

- Test conclusions depend on the chosen beta-diversity metric (Bray-Curtis, UniFrac, Jaccard); the review emphasizes selecting appropriate methods and summarizes their advantages and limitations
- PERMANOVA tests only whether groups differ in overall community structure — it does not identify which features are biomarkers of each group, which requires separate difference-testing
- Results assume the feature table and metadata are correctly paired; the article notes tool/pipeline selection is challenging for researchers without a bioinformatics background, so misapplied statistical choices are a real failure mode

## Evidence

- [abstract] Beta-diversity group testing context: "we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity, taxonomic composition, difference comparisons, correlation, networks, machine"
- [abstract] Reproducible step-by-step workflow: "Finally, a step-by-step reproducible analysis guide is introduced."
- [intro] Statistical analysis coverage in review: "In this review, we discuss the widely used software packages for microbiome analyses, summarize their advantages and limitations, and provide sample codes and suggestions for selecting and using"
