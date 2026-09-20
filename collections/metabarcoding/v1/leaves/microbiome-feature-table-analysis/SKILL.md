---
name: microbiome-feature-table-analysis
description: Use when an amplicon feature table (BIOM/QIIME 2 format) and a sample
  metadata mapping file are loaded and you must determine whether experimental groups
  differ in alpha- or beta-diversity and which features are biomarkers of each group.
when_to_use_negative:
- Input is still raw sequencing reads that have not been denoised into a feature table
  (run the amplicon pipeline first before diversity statistics)
- The goal is functional metagenome prediction rather than diversity/composition statistics
  (PICRUSt2 predicts metagenomes from amplicon data)
- Tree and annotation visualization is the sole goal (table2itol generates iTOL annotation
  files, not diversity tests)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3053
tools:
- name: QIIME 2
  role: Reproducible, interactive, community-supported microbiome analysis pipeline
    from which feature tables are produced and analyzed
- name: vegan
  role: R package implementing beta-diversity ordination (PCoA, NMDS, CPCoA) and PERMANOVA
    (adonis) for group-separation testing
- name: mgoeker/table2itol
  role: R script generating iTOL annotation files (colour strips, bars, heat maps)
    from spreadsheet tables for microbiome result visualization
  repo: https://github.com/mgoeker/table2itol
- name: YongxinLiu/Liu2020ProteinCell
  role: Pipeline and reproducible analysis examples for amplicon and metagenome microbiome
    analysis accompanying the paper
  repo: https://github.com/YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:e62cd5f9735a2462f2920d64b8f7f3066ee967928f97e15ccacefab09e714105
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mgoeker/table2itol
  tools:
  - QIIME 2
  - YongxinLiu/Liu2020ProteinCell
  - mgoeker/table2itol
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# microbiome-feature-table-analysis

## Summary

Statistical and visualization analysis of microbiome amplicon feature tables: computing alpha/beta diversity, testing group differences (ANOVA, Mann-Whitney U, Kruskal-Wallis, PERMANOVA), ordinating samples (PCoA, NMDS, CPCoA), and assembling comparison figures and tables.

## When to use

Use when an amplicon feature table (BIOM/QIIME 2 format) and a sample metadata mapping file are loaded and you must determine whether experimental groups differ in alpha- or beta-diversity and which features are biomarkers of each group.

## When NOT to use

- Input is still raw sequencing reads that have not been denoised into a feature table (run the amplicon pipeline first before diversity statistics)
- The goal is functional metagenome prediction rather than diversity/composition statistics (PICRUSt2 predicts metagenomes from amplicon data)
- Tree and annotation visualization is the sole goal (table2itol generates iTOL annotation files, not diversity tests)

## Inputs

- Amplicon feature table in BIOM or QIIME 2 format
- Sample metadata mapping file
- Beta-diversity distance matrices (Bray-Curtis, UniFrac, Jaccard)

## Outputs

- Alpha-diversity boxplots per group with test statistics and p-values
- Beta-diversity ordination scatter plots (PCoA, NMDS, CPCoA)
- PERMANOVA results on the distance matrix
- Group-difference comparison table

## How to apply

Load the BIOM/QIIME 2 feature table and metadata mapping file into R, compute per-sample alpha-diversity indices, and test group differences with ANOVA (parametric), Mann-Whitney U (two groups), or Kruskal-Wallis (multiple groups). Compute beta-diversity distance matrices between samples using Bray-Curtis, UniFrac, or Jaccard, then ordinate samples with PCoA, NMDS, or constrained CPCoA implemented in the R vegan package. Assess group separation with PERMANOVA (vegan adonis) on the distance matrix, and draw alpha-diversity boxplots and beta-diversity ordination scatter plots with ggplot2. Assemble the group-difference comparison figure and table, verifying that test statistics and p-values are consistent with the metadata group assignments.

## Related tools

- **QIIME 2** (Reproducible, interactive, community-supported microbiome analysis pipeline from which feature tables are produced and analyzed)
- **vegan** (R package implementing beta-diversity ordination (PCoA, NMDS, CPCoA) and PERMANOVA (adonis) for group-separation testing)
- **mgoeker/table2itol** (R script generating iTOL annotation files (colour strips, bars, heat maps) from spreadsheet tables for microbiome result visualization) — https://github.com/mgoeker/table2itol
- **YongxinLiu/Liu2020ProteinCell** (Pipeline and reproducible analysis examples for amplicon and metagenome microbiome analysis accompanying the paper) — https://github.com/YongxinLiu/Liu2020ProteinCell

## Examples

```
library(vegan); adonis(beta_dist_matrix ~ group, data = metadata, permutations = 999)
```

## Evaluation signals

- Alpha-diversity test statistics and p-values are generated for every metadata group comparison (ANOVA, Mann-Whitney U, or Kruskal-Wallis as appropriate to group count)
- Beta-diversity distance matrix (Bray-Curtis, UniFrac, or Jaccard) is computed before ordination and PERMANOVA is run on it
- Ordination scatter plots and alpha-diversity boxplots are generated as figure files consistent with the metadata group assignments
- Difference-comparison table values agree with the computed test statistics and sample group labels
- All figure files and tables are reproducible via the step-by-step pipeline in the Liu2020ProteinCell repository

## Limitations

- Researchers without a bioinformatics background face challenges choosing suitable software and pipelines among the many new developments
- Parametric ANOVA is only appropriate when its assumptions hold; non-parametric alternatives (Mann-Whitney U, Kruskal-Wallis) must be chosen for two-group or multi-group comparisons otherwise
- Metatranscriptome, metaproteome, and metabolome analysis methods are not widely accepted due to high cost and complex experimental and analysis methods

## Evidence

- [abstract] we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity, taxonomic composition, difference comparisons, correlation, networks, machine: "we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity, taxonomic composition, difference comparisons, correlation, networks, machine"
- [abstract] Finally, a step-by-step reproducible analysis guide is introduced.: "Finally, a step-by-step reproducible analysis guide is introduced."
- [readme] Pipeline & Reproducible analysis examples for amplicon and metagenome in microbiome: "Pipeline & Reproducible analysis examples for amplicon and metagenome in microbiome"
