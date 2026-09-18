---
name: nonparametric-hypothesis-testing
description: Use when comparing alpha- or beta-diversity across experimental groups
  in an amplicon feature table (BIOM/QIIME 2) plus metadata mapping file, to test
  group differences statistically.
when_to_use_negative:
- Input is raw sequencing reads still awaiting the QIIME 2 amplicon pipeline (denoising/ASV
  inference) rather than a finished feature table
- The goal is predicting functional potential from 16S data — use PICRUSt2 (picrust/picrust2)
  instead of diversity testing
- Parametric assumptions (e.g., normality for ANOVA) demonstrably hold and the reviewer
  explicitly requires a parametric framework
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3697
tools:
- name: vegan
  role: R package used for ordination (PCoA, NMDS, CPCoA) and PERMANOVA (adonis) on
    beta-diversity distance matrices
- name: QIIME 2
  role: Produces the amplicon feature table (BIOM/QIIME 2 format) and supports alpha/beta-diversity
    computation that feeds the group-difference tests
- name: ggplot2
  role: Draws alpha-diversity boxplots and beta-diversity ordination scatter plots
    summarizing test results
- name: Liu2020ProteinCell
  role: Repository with the reproducible amplicon/metagenome pipeline and analysis
    examples underlying this workflow
  repo: YongxinLiu/Liu2020ProteinCell
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s13238-020-00724-8
    title: A practical guide to amplicon and metagenomic analysis of microbiome data
  source_content_hash: sha256:fcfe165e96da0e74abfb8ea342254bfffaacb5abed820b369f3b168de80e399e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YongxinLiu/Liu2020ProteinCell
  tools:
  - Liu2020ProteinCell
  - QIIME 2
  - ggplot2
  - vegan
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# nonparametric-hypothesis-testing

## Summary

Applies nonparametric hypothesis testing to microbiome diversity data: Mann-Whitney U or Kruskal-Wallis tests on alpha-diversity indices and PERMANOVA (vegan adonis) on beta-diversity distance matrices to determine whether experimental groups differ significantly.

## When to use

Use when comparing alpha- or beta-diversity across experimental groups in an amplicon feature table (BIOM/QIIME 2) plus metadata mapping file, to test group differences statistically.

## When NOT to use

- Input is raw sequencing reads still awaiting the QIIME 2 amplicon pipeline (denoising/ASV inference) rather than a finished feature table
- The goal is predicting functional potential from 16S data — use PICRUSt2 (picrust/picrust2) instead of diversity testing
- Parametric assumptions (e.g., normality for ANOVA) demonstrably hold and the reviewer explicitly requires a parametric framework

## Inputs

- Amplicon feature table in BIOM or QIIME 2 format
- Sample metadata mapping file with group assignments
- Beta-diversity distance matrix (Bray-Curtis, UniFrac, or Jaccard)
- Alpha-diversity index values per sample

## Outputs

- Alpha-diversity group-difference test statistics and p-values (Mann-Whitney U, Kruskal-Wallis, or ANOVA)
- PERMANOVA (adonis) statistics and p-values for beta-diversity group separation
- Alpha-diversity boxplots (ggplot2)
- Beta-diversity ordination scatter plots (PCoA/NMDS/CPCoA)
- Difference-comparison table of group differences

## How to apply

Load the amplicon feature table (BIOM/QIIME 2 format) and sample metadata mapping file into R. Compute alpha-diversity indices per sample and test group differences nonparametrically: Mann-Whitney U for two groups, Kruskal-Wallis for multiple groups (ANOVA is the parametric alternative when assumptions hold), and visualize results with boxplots (ggplot2). For beta diversity, compute a distance matrix between samples (Bray-Curtis, UniFrac, or Jaccard), ordinate samples with PCoA, NMDS, or constrained CPCoA from the R vegan package, then assess group separation with PERMANOVA via vegan's adonis function. A parametric ANOVA is acceptable only when its assumptions are met; otherwise the nonparametric tests are preferred. Results should be assembled with the alpha-diversity boxplots, beta-diversity ordination scatter plots, and a difference-comparison table.

## Related tools

- **vegan** (R package used for ordination (PCoA, NMDS, CPCoA) and PERMANOVA (adonis) on beta-diversity distance matrices)
- **QIIME 2** (Produces the amplicon feature table (BIOM/QIIME 2 format) and supports alpha/beta-diversity computation that feeds the group-difference tests)
- **ggplot2** (Draws alpha-diversity boxplots and beta-diversity ordination scatter plots summarizing test results)
- **Liu2020ProteinCell** (Repository with the reproducible amplicon/metagenome pipeline and analysis examples underlying this workflow) — YongxinLiu/Liu2020ProteinCell

## Examples

```
adonis(dist_matrix ~ group, data = metadata, permutations = 999)  # vegan::adonis PERMANOVA on Bray-Curtis/UniFrac/Jaccard distances
```

## Evaluation signals

- Test statistics and p-values for the alpha/beta-diversity tests are generated and consistent with the metadata group assignments
- PERMANOVA is run on a named distance matrix (Bray-Curtis, UniFrac, or Jaccard) rather than raw abundances
- Alpha-diversity boxplots and beta-diversity ordination scatter plot files are produced and match the tested groups
- The difference-comparison table is present and its group labels agree with the mapping file
- Two-group comparisons use Mann-Whitney U while multi-group comparisons use Kruskal-Wallis (or justified parametric ANOVA)

## Limitations

- Nonparametric tests do not themselves identify which taxonomic features are biomarkers of each group; a separate difference-comparison method is needed
- ANOVA is an option only under parametric assumptions; applying it blindly when assumptions fail can mislead
- Beta-diversity conclusions depend on the chosen distance metric (Bray-Curtis, UniFrac, Jaccard) — different metrics can yield different PERMANOVA results

## Evidence

- [other] alpha/beta tests via source card finding: [evidence span withheld — anchor status: not_found]
- [other] PERMANOVA on distance matrix: [evidence span withheld — anchor status: not_found]
- [abstract] article scope statement: "we introduce statistical and visualization methods suitable for microbiome analysis, including alpha- and beta-diversity, taxonomic composition, difference comparisons, correlation, networks, machine"
- [abstract] reproducible workflow: "Finally, a step-by-step reproducible analysis guide is introduced."
