---
name: permanova-effect-size-estimation
description: Use when comparing differential abundance tools across multiple 16S rRNA
  datasets and a dataset-level effect size of between-group community differences
  (e.g., via PERMANOVA on Aitchison distance) is needed as a characteristic to correlate
  with per-tool numbers of significant ASVs.
when_to_use_negative:
- The dataset is a single metagenome/16S dataset being analyzed for DA features, not
  a multi-dataset comparison — a dataset-level effect size adds nothing to per-ASV
  testing.
- Samples lack a two-group design (e.g., longitudinal or >2 groups), so the Aitchison-distance
  PERMANOVA between test groups used here is not applicable.
- The goal is consistency across datasets of the same environment (obesity/diarrhea
  analyses) rather than correlating significance rates with dataset characteristics.
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: vegan
  role: R package used to compute Spearman rank correlations and support PERMANOVA/Aitchison
    distance analysis in the correlation workflow
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: dplyr
  role: R package (v0.8.5) used to compute the percent of significant ASVs per tool
    per dataset before correlation
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: tidyverse
  role: R environment (with ggplot2 v3.3.0) used to assemble the tool x characteristic
    correlation table and reproduce the Fig. 2 relationships
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: compositions
  role: R package (v1.40.2) providing the compositional (Aitchison) data transforms
    underlying the Aitchison distance used for PERMANOVA
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:e1c29ed5b19bd9ccd516c51395b81af6a9fcac416061ebceb101f5f7f094d7c1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - compositions
  - dplyr
  - tidyverse
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# permanova-effect-size-estimation

## Summary

Compute a per-dataset PERMANOVA effect size on the Aitchison distance between sample groups, and use it as the dataset characteristic against which the percent of significant ASVs from each differential abundance tool is Spearman-correlated. In Nearing et al. 2022 this effect size is stored in aitchison_permanova_results.tsv for the 38 16S rRNA datasets and underlies the Fig. 2 correlation analysis.

## When to use

Use when comparing differential abundance tools across multiple 16S rRNA datasets and a dataset-level effect size of between-group community differences (e.g., via PERMANOVA on Aitchison distance) is needed as a characteristic to correlate with per-tool numbers of significant ASVs.

## When NOT to use

- The dataset is a single metagenome/16S dataset being analyzed for DA features, not a multi-dataset comparison — a dataset-level effect size adds nothing to per-ASV testing.
- Samples lack a two-group design (e.g., longitudinal or >2 groups), so the Aitchison-distance PERMANOVA between test groups used here is not applicable.
- The goal is consistency across datasets of the same environment (obesity/diarrhea analyses) rather than correlating significance rates with dataset characteristics.

## Inputs

- 16S rRNA ASV count table (TSV) and metadata file with two sample groups per dataset (38 datasets)
- Per-tool significant ASV outputs from run_no_filt_datasets.sh (unfiltered datasets)
- Per-tool significant ASV outputs from run_0.1_filt_datasets.sh (10% prevalence-filtered datasets)
- aitchison_permanova_results.tsv (PERMANOVA results on Aitchison distance from Misc_datafiles)

## Outputs

- Per-dataset PERMANOVA effect size table on Aitchison distance (aitchison_permanova_results.tsv-style)
- Dataset characteristic table (PERMANOVA effect size, sample size, median/range read depth, richness, sparsity)
- spearman_correlations.csv: tool x characteristic Spearman rho table for unfiltered and 10%-filtered conditions

## How to apply

For each of the 38 two-group 16S rRNA datasets, compute the Aitchison distance between samples and run PERMANOVA to obtain an effect size quantifying the strength of between-group community differences; store the result per dataset in a table like aitchison_permanova_results.tsv. Then join this effect size with the other dataset characteristics (sample size, median/range read depth, richness, sparsity) and, for each DA tool, compute the percent of significant ASVs per dataset with dplyr from the outputs of run_no_filt_datasets.sh (unfiltered) and run_0.1_filt_datasets.sh (10% prevalence filter). Compute Spearman rank correlations with cor(method='spearman') in R (vegan/tidyverse) between each tool's percent significant ASVs and the effect size, separately for unfiltered and filtered data, and assemble the tool x characteristic coefficient table into spearman_correlations.csv. The rationale is that the article reports the number of features identified by DA tools correlates with 'the effect size of community differences', so effect size is the key continuous predictor of tool significance rates.

## Related tools

- **vegan** (R package used to compute Spearman rank correlations and support PERMANOVA/Aitchison distance analysis in the correlation workflow) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **dplyr** (R package (v0.8.5) used to compute the percent of significant ASVs per tool per dataset before correlation) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **tidyverse** (R environment (with ggplot2 v3.3.0) used to assemble the tool x characteristic correlation table and reproduce the Fig. 2 relationships) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **compositions** (R package (v1.40.2) providing the compositional (Aitchison) data transforms underlying the Aitchison distance used for PERMANOVA) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
cor(percent_sig_asvs$tool, characteristics$permanova_effect_size, method = "spearman")
```

## Evaluation signals

- Unfiltered Spearman coefficients between percent significant ASVs and effect size fall within the reported 0.35–0.72 range across tools.
- Filtered (10% prevalence filter) Spearman coefficients fall within the reported 0.31–0.56 range across tools.
- The tool x characteristic correlation table reproduces the positive correlation relationship shown in Fig. 2.
- The effect size column matches aitchison_permanova_results.tsv for all 38 datasets.
- spearman_correlations.csv contains separate coefficient sets for unfiltered and filtered conditions.

## Limitations

- The 10% prevalence filter cut-off was arbitrarily selected; more work is needed to estimate an optimal cut-off.
- Correlation with effect size does not indicate which tool is correct — the authors could not make a direct simple recommendation of a specific tool from the analysis.
- Results depend on data pre-processing, so unfiltered and filtered pipelines must be run and reported separately.

## Evidence

- [readme] PERMANOVA effect size on Aitchison distance stored per dataset: [evidence span withheld — anchor status: not_found]
- [abstract] Effect size correlates with numbers of significant features found by tools: "the number of features identified correlate with aspects of the data, such as sample size, sequencing depth, and effect size of community differences"
- [discussion] Prevalence filtering condition under which correlations are recomputed: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing"
- [readme] Filtered and unfiltered dataset pipelines whose outputs feed the correlation analysis: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
