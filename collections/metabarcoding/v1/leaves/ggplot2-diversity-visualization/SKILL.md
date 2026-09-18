---
name: ggplot2-diversity-visualization
description: Use when an amplicon feature table (BIOM/QIIME 2 format) with a sample
  metadata mapping file needs group-difference testing and figure/table output for
  alpha- and beta-diversity in R.
when_to_use_negative:
- The input is raw sequencing data rather than an already-processed amplicon feature
  table — feature-table generation is upstream of this skill
- The goal is functional potential prediction from amplicon data (e.g. PICRUSt2 pathway
  inference) rather than diversity statistics and visualization
- The goal is interactive iTOL tree annotation from spreadsheets (table2itol use case),
  not diversity testing
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_3061
tools:
- name: vegan
  role: R package used to compute beta-diversity distance matrices, perform PCoA/NMDS/CPCoA
    ordination, and run PERMANOVA group-separation testing via adonis
- name: ggplot2
  role: R package used to draw the alpha-diversity boxplots and beta-diversity ordination
    scatter plots for the group-difference figure
- name: QIIME 2
  role: Pipeline whose BIOM/QIIME 2 feature-table output is the input data loaded
    into R for diversity testing and visualization
- name: Liu2020ProteinCell
  role: Repository providing the pipeline and reproducible analysis examples for amplicon
    and metagenome diversity analysis demonstrated in the review
  repo: YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:63fbef0065aef83cc0f3f23ed1c6a72943f6472e530925163e0b31707bd992cd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - Liu2020ProteinCell
  - QIIME 2
  - ggplot2
  - vegan
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_3061
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# ggplot2-diversity-visualization

## Summary

Statistical testing and visualization of microbiome alpha- and beta-diversity in R: alpha diversity is compared with ANOVA, Mann-Whitney U or Kruskal-Wallis and shown as ggplot2 boxplots, while beta diversity is assessed with PERMANOVA on Bray-Curtis/UniFrac/Jaccard distance matrices and visualized as PCoA/NMDS ordination scatter plots.

## When to use

Use when an amplicon feature table (BIOM/QIIME 2 format) with a sample metadata mapping file needs group-difference testing and figure/table output for alpha- and beta-diversity in R.

## When NOT to use

- The input is raw sequencing data rather than an already-processed amplicon feature table — feature-table generation is upstream of this skill
- The goal is functional potential prediction from amplicon data (e.g. PICRUSt2 pathway inference) rather than diversity statistics and visualization
- The goal is interactive iTOL tree annotation from spreadsheets (table2itol use case), not diversity testing

## Inputs

- Amplicon feature table in BIOM or QIIME 2 format
- Sample metadata mapping file assigning samples to experimental groups
- Optional phylogenetic tree (for UniFrac beta diversity)

## Outputs

- Alpha-diversity per-sample index values with group-difference test statistics and p-values (ANOVA / Mann-Whitney U / Kruskal-Wallis)
- Beta-diversity distance matrices (Bray-Curtis, UniFrac, Jaccard) with PERMANOVA (adonis) results
- Ordination coordinates from PCoA, NMDS, or CPCoA
- Alpha-diversity boxplots and beta-diversity ordination scatter plots (ggplot2 figure files)
- Group-difference comparison table

## How to apply

Load the amplicon feature table (BIOM or QIIME 2 format) and sample metadata mapping file into R. Compute per-sample alpha-diversity indices and choose the test by design: ANOVA for parametric comparisons, Mann-Whitney U for two groups, Kruskal-Wallis for multiple groups, then draw the group comparison as a ggplot2 boxplot. Compute beta-diversity distance matrices between samples using Bray-Curtis, UniFrac, or Jaccard, and ordinate samples with PCoA, NMDS, or constrained CPCoA from the vegan package. Test group separation with PERMANOVA (vegan adonis) on the distance matrix, and render the ordination as a ggplot2 scatter plot colored by metadata group. Assemble the group-difference comparison figure and table, confirming that test statistics, p-values, and group assignments are consistent with the metadata.

## Related tools

- **vegan** (R package used to compute beta-diversity distance matrices, perform PCoA/NMDS/CPCoA ordination, and run PERMANOVA group-separation testing via adonis)
- **ggplot2** (R package used to draw the alpha-diversity boxplots and beta-diversity ordination scatter plots for the group-difference figure)
- **QIIME 2** (Pipeline whose BIOM/QIIME 2 feature-table output is the input data loaded into R for diversity testing and visualization)
- **Liu2020ProteinCell** (Repository providing the pipeline and reproducible analysis examples for amplicon and metagenome diversity analysis demonstrated in the review) — YongxinLiu/Liu2020ProteinCell

## Examples

```
library(vegan); adonis(bray_curtis_dist ~ group, data = metadata, permutations = 999)
```

## Evaluation signals

- Alpha/beta-diversity test statistics and p-values are generated and consistent with the metadata group assignments
- All figure files (alpha-diversity boxplots, beta-diversity ordination scatter plots) are produced and colored/matched to the correct metadata groups
- The group-difference comparison table is generated and agrees with the reported PERMANOVA and alpha-diversity test results
- Ordination output is one of the article-recommended methods (PCoA, NMDS, or constrained CPCoA from vegan) and the distance matrix is Bray-Curtis, UniFrac, or Jaccard
- The alpha-diversity test matches the design: Mann-Whitney U for two groups, Kruskal-Wallis or ANOVA for multiple groups

## Limitations

- Choice of statistical test is a decision point: parametric ANOVA assumes appropriate distributional conditions, whereas Mann-Whitney U (two groups) and Kruskal-Wallis (multiple groups) are the non-parametric alternatives
- The beta-diversity result depends on the chosen distance metric (Bray-Curtis, UniFrac, Jaccard); the choice should be justified per analysis
- Ordination is descriptive — significance of group separation requires a formal PERMANOVA (vegan adonis) test on the distance matrix rather than visual inspection alone

## Evidence

- [abstract] Statistical and visualization methods for diversity comparison: "we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity, taxonomic composition, difference comparisons, correlation, networks, machine"
- [abstract] Step-by-step reproducible analysis guide: "Finally, a step-by-step reproducible analysis guide is introduced."
- [discussion] Scope of the review across the analysis stages: "from the selection of sequencing methods, analysis software/pipelines, statistical analysis and visualization to the implementation of reproducible analysis"
