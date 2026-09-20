---
name: beta-diversity-ordination-pcoa
description: Use when amplicon feature tables (BIOM/QIIME 2) plus a sample metadata
  mapping file are available and the goal is to test whether experimental groups differ
  in community composition and visualize sample separation on ordination plots.
when_to_use_negative:
- The question concerns within-sample (alpha) diversity only — use ANOVA, Mann-Whitney
  U, or Kruskal-Wallis on alpha-diversity indices with boxplots instead
- Input is raw sequencing data that has not yet been reduced to a feature table (BIOM/QIIME
  2) — run the upstream amplicon pipeline first
- The goal is functional potential prediction rather than community-composition comparison
  — PICRUSt2 applies to that task
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0610
tools:
- name: vegan
  role: R package used for ordination (PCoA, NMDS, constrained CPCoA) and for PERMANOVA
    testing of group differences via adonis on the beta-diversity distance matrix
- name: ggplot2
  role: Used to draw beta-diversity ordination scatter plots colored by metadata group
- name: QIIME 2
  role: Source pipeline whose feature-table (BIOM/QIIME 2) outputs and sample metadata
    are loaded into R for beta-diversity analysis
- name: YongxinLiu/Liu2020ProteinCell
  role: Reproducible pipeline and analysis examples for the amplicon workflow this
    skill follows
  repo: https://github.com/YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:f769f1142d03e48d758fb2990736c7621e9647ae0138d55d9fdb52427e11b49c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - QIIME 2
  - YongxinLiu/Liu2020ProteinCell
  - ggplot2
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Beta-diversity ordination and group-difference testing (PCoA/NMDS + PERMANOVA)

## Summary

Computes beta-diversity distance matrices between samples (Bray-Curtis, UniFrac, Jaccard), ordinated them with PCoA/NMDS/CPCoA, and evaluates whether experimental groups differ using PERMANOVA, following the amplicon-analysis pipeline of Liu et al. 2020.

## When to use

Use when amplicon feature tables (BIOM/QIIME 2) plus a sample metadata mapping file are available and the goal is to test whether experimental groups differ in community composition and visualize sample separation on ordination plots.

## When NOT to use

- The question concerns within-sample (alpha) diversity only — use ANOVA, Mann-Whitney U, or Kruskal-Wallis on alpha-diversity indices with boxplots instead
- Input is raw sequencing data that has not yet been reduced to a feature table (BIOM/QIIME 2) — run the upstream amplicon pipeline first
- The goal is functional potential prediction rather than community-composition comparison — PICRUSt2 applies to that task

## Inputs

- Amplicon feature table in BIOM or QIIME 2 format
- Sample metadata mapping file defining experimental groups

## Outputs

- Beta-diversity distance matrices (Bray-Curtis, UniFrac, or Jaccard)
- Ordination scatter plots (PCoA/NMDS/CPCoA sample coordinates colored by group)
- PERMANOVA (vegan adonis) test statistics and p-values for group differences
- Group-difference comparison figure and table

## How to apply

Load the amplicon feature table (BIOM/QIIME 2 format) and the sample metadata mapping file into R, then compute a beta-diversity distance matrix between samples using Bray-Curtis, UniFrac, or Jaccard metrics chosen for the research question. Ordinate the distance matrix with PCoA, NMDS, or constrained CPCoA as implemented in the R vegan package to obtain low-dimensional sample coordinates. Test whether metadata groups separate on the distance matrix with PERMANOVA (vegan adonis), which partitions variance explained by the group factor and returns a p-value from permutations. Draw beta-diversity ordination scatter plots colored by metadata group with ggplot2 and assemble them into the group-difference comparison figure alongside the PERMANOVA statistic and p-value.

## Related tools

- **vegan** (R package used for ordination (PCoA, NMDS, constrained CPCoA) and for PERMANOVA testing of group differences via adonis on the beta-diversity distance matrix)
- **ggplot2** (Used to draw beta-diversity ordination scatter plots colored by metadata group)
- **QIIME 2** (Source pipeline whose feature-table (BIOM/QIIME 2) outputs and sample metadata are loaded into R for beta-diversity analysis)
- **YongxinLiu/Liu2020ProteinCell** (Reproducible pipeline and analysis examples for the amplicon workflow this skill follows) — https://github.com/YongxinLiu/Liu2020ProteinCell

## Examples

```
R: pcoa <- cmdscale(bray_dist, k = 2); adonis(bray_dist ~ group, data = meta, permutations = 999)
```

## Evaluation signals

- Distance matrices are computed for all sample pairs (e.g. Bray-Curtis/UniFrac/Jaccard) and ordination returns 2-D sample coordinates that reproduce the distance structure
- PERMANOVA via vegan adonis returns a test statistic and permutation p-value consistent with the metadata group assignments
- Ordination scatter plots show samples colored/labeled according to the metadata groups, with points separated if PERMANOVA is significant
- The difference-comparison table contains the beta-diversity test statistics and p-values and is consistent with the figure files
- All figure files and the difference-comparison table are generated and cross-checked against the feature table and mapping file

## Limitations

- PERMANOVA assesses whether groups differ in centroid/dispersion on the chosen distance matrix; the result depends on which metric (Bray-Curtis, UniFrac, Jaccard) is selected
- Ordination is exploratory — significant visual separation should be backed by the PERMANOVA test rather than judged from the scatter plot alone
- Metatranscriptome, metaproteome, and metabolome analyses are out of scope; the article notes these methods 'have not been widely accepted due to their high cost and the complex experimental and analysis methods required'

## Evidence

- [other] Beta-diversity differences assessed with PERMANOVA and ordination: [evidence span withheld — anchor status: not_found]
- [other] Ordination tools in vegan and PERMANOVA via adonis: [evidence span withheld — anchor status: not_found]
- [abstract] Article scope covers alpha/beta diversity statistical and visualization methods: "we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity, taxonomic composition, difference comparisons, correlation, networks, machine"
- [abstract] Reproducible step-by-step amplicon pipeline guidance: "Finally, a step-by-step reproducible analysis guide is introduced."
