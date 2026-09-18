---
name: alpha-diversity-group-comparison
description: Use when an amplicon feature table (BIOM or QIIME 2 format) plus a sample
  metadata mapping file must be tested for significant group differences in alpha-diversity
  indices, with boxplot figures as output.
when_to_use_negative:
- The question is about beta-diversity group separation, which requires distance matrices,
  ordination (PCoA/NMDS/CPCoA), and PERMANOVA (vegan adonis) rather than per-sample
  alpha tests
- Input is raw sequencing reads rather than an already-processed feature table; upstream
  amplicon pipeline steps must run first
- The goal is taxonomic composition or differential-feature (biomarker) testing rather
  than within-sample diversity comparison
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0092
tools:
- name: QIIME 2
  role: Source pipeline for the amplicon feature table (BIOM/QIIME 2 format) and sample
    metadata consumed by the alpha-diversity tests
- name: vegan
  role: R package used to compute diversity statistics on the feature table as part
    of the reproducible analysis workflow
- name: ggplot2
  role: R plotting library used to draw the alpha-diversity boxplots comparing groups
- name: YongxinLiu/Liu2020ProteinCell
  role: Repository with the pipeline and reproducible analysis examples for the amplicon
    alpha/beta-diversity workflow described in this review
  repo: YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:dd31169fff94956123e5108a3647e9baef8c4eeec50c44492161d9422df40643
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - QIIME 2
  - YongxinLiu/Liu2020ProteinCell
  - ggplot2
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# alpha-diversity-group-comparison

## Summary

Tests whether experimental groups differ in alpha-diversity indices computed from an amplicon feature table (BIOM/QIIME 2 format), using ANOVA, Mann-Whitney U, or Kruskal-Wallis tests and visualizing per-sample values as boxplots.

## When to use

Use when an amplicon feature table (BIOM or QIIME 2 format) plus a sample metadata mapping file must be tested for significant group differences in alpha-diversity indices, with boxplot figures as output.

## When NOT to use

- The question is about beta-diversity group separation, which requires distance matrices, ordination (PCoA/NMDS/CPCoA), and PERMANOVA (vegan adonis) rather than per-sample alpha tests
- Input is raw sequencing reads rather than an already-processed feature table; upstream amplicon pipeline steps must run first
- The goal is taxonomic composition or differential-feature (biomarker) testing rather than within-sample diversity comparison

## Inputs

- Amplicon feature table (BIOM or QIIME 2 format)
- Sample metadata mapping file with group assignments

## Outputs

- Per-sample alpha-diversity index table
- Alpha-diversity boxplot figures (ggplot2)
- Difference-comparison table with test statistics and p-values (ANOVA / Mann-Whitney U / Kruskal-Wallis)

## How to apply

Load the amplicon feature table (BIOM/QIIME 2 format) and the sample metadata mapping file into R, then compute alpha-diversity indices per sample. Choose the test by design: Mann-Whitney U for two groups, Kruskal-Wallis (non-parametric) or ANOVA (parametric) for multiple groups, reflecting the review's guidance that these tests statistically evaluate alpha-diversity differences among or between groups. Run the selected test on each alpha-diversity index against the metadata grouping factor and record test statistics and p-values in a difference-comparison table. Draw alpha-diversity boxplots with ggplot2, with samples colored by group, and assemble the figure and table. Validate that all test statistics, p-values, figure files, and the comparison table are generated and consistent with the metadata group assignments.

## Related tools

- **QIIME 2** (Source pipeline for the amplicon feature table (BIOM/QIIME 2 format) and sample metadata consumed by the alpha-diversity tests)
- **vegan** (R package used to compute diversity statistics on the feature table as part of the reproducible analysis workflow)
- **ggplot2** (R plotting library used to draw the alpha-diversity boxplots comparing groups)
- **YongxinLiu/Liu2020ProteinCell** (Repository with the pipeline and reproducible analysis examples for the amplicon alpha/beta-diversity workflow described in this review) — YongxinLiu/Liu2020ProteinCell

## Examples

```
kruskal.test(alpha_index ~ group, data = alpha_metrics)
```

## Evaluation signals

- Test statistics and p-values for each alpha-diversity index are present in the difference-comparison table and match the metadata group assignments
- The statistical test matches the design: Mann-Whitney U for two groups, ANOVA or Kruskal-Wallis for multiple groups
- Boxplot figures show per-sample alpha-diversity values grouped by the metadata factor, with one panel per index
- All figure files, the comparison table, and the alpha/beta statistics are regenerated reproducibly by the pipeline scripts
- Group labels in the outputs are consistent with the sample metadata mapping file (no sample-to-group mismatches)

## Limitations

- ANOVA is parametric; group distributions of alpha-diversity indices must satisfy its assumptions, otherwise use Mann-Whitney U or Kruskal-Wallis
- The choice between Mann-Whitney U, Kruskal-Wallis, and ANOVA depends on the number of groups being compared; an inappropriate test weakens the comparison
- The article notes that it can be challenging for researchers, especially those without a bioinformatics background, to choose suitable software and pipelines for these analyses

## Evidence

- [abstract] we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity: "we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity"
- [other] Differences in alpha diversity among or between groups can be statistically evaluated with ANOVA, Mann-Whitney U test, or Kruskal-Wallis test, visualized via boxplots: [evidence span withheld — anchor status: not_found]
- [intro] In this review, we discuss the widely used software packages for microbiome analyses, summarize their advantages and limitations, and provide sample codes and suggestions for selecting and using: "In this review, we discuss the widely used software packages for microbiome analyses, summarize their advantages and limitations, and provide sample codes and suggestions for selecting and using"
