---
name: microbiome-dataset-characteristic-computation
description: Use when you have per-tool significant ASV outputs for many two-group
  16S datasets and want to test whether the numbers of significant features correlate
  with effect size, sample size, sequencing depth, or sparsity, with vs without prevalence
  filtering.
when_to_use_negative:
- 'You have not yet run the DA tools: this skill consumes per-tool significant ASV
  outputs, not raw ASV tables — run run_all_tools.sh first'
- Your question is tool-to-tool agreement (consistency/intersect of significant features),
  not correlation of significance counts with dataset characteristics
- Datasets lack two sample groups, so a PERMANOVA effect size on Aitchison distance
  between test groups cannot be computed
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: vegan
  role: R package used for computing PERMANOVA effect sizes on Aitchison distance
    and supporting the correlation analysis
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: dplyr
  role: Used to compute the percent of significant ASVs per tool per dataset (version
    0.8.5 listed in repo requirements)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: tidyverse
  role: Used for data assembly and Spearman correlation table construction
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:f5d9bb9559187e03037f66e96aecb49c63166471fe52af8b9fe22ce103acbc8d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - dplyr
  - tidyverse
  - vegan
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# microbiome-dataset-characteristic-computation

## Summary

Assembles per-dataset characteristics (Aitchison PERMANOVA effect size, sample size, read depth, richness, sparsity) for 16S rRNA gene datasets and correlates them against each differential abundance tool's percent of significant ASVs using Spearman rank correlation, under unfiltered and 10% prevalence-filtered conditions.

## When to use

Use when you have per-tool significant ASV outputs for many two-group 16S datasets and want to test whether the numbers of significant features correlate with effect size, sample size, sequencing depth, or sparsity, with vs without prevalence filtering.

## When NOT to use

- You have not yet run the DA tools: this skill consumes per-tool significant ASV outputs, not raw ASV tables — run run_all_tools.sh first
- Your question is tool-to-tool agreement (consistency/intersect of significant features), not correlation of significance counts with dataset characteristics
- Datasets lack two sample groups, so a PERMANOVA effect size on Aitchison distance between test groups cannot be computed

## Inputs

- Per-tool significant ASV result files for the 38 unfiltered datasets (output of run_no_filt_datasets.sh)
- Per-tool significant ASV result files for the 38 filtered datasets (output of run_0.1_filt_datasets.sh, 10% prevalence filter)
- aitchison_permanova_results.tsv (PERMANOVA effect sizes on Aitchison distance, from Misc_datafiles)
- Per-dataset metadata/mapping files with sample size and read depth (16S rRNA Microbiome Datasets from figshare)

## Outputs

- spearman_correlations.csv: tool x characteristic Spearman rho table for both unfiltered and filtered conditions
- Per-dataset percent significant ASVs per tool (dplyr-derived summary table)
- Dataset characteristic table (PERMANOVA effect size, sample size, median/range read depth, richness, sparsity)

## How to apply

Load the per-tool significant ASV results produced by run_no_filt_datasets.sh (38 unfiltered datasets) and run_0.1_filt_datasets.sh (38 filtered datasets with a 10% prevalence filter, i.e. -F 0.1). For each dataset compute the percent of significant ASVs per tool with dplyr. Build the characteristic table from aitchison_permanova_results.tsv (PERMANOVA effect size on Aitchison distance) plus sample size, median/range read depth, richness, and sparsity per dataset. Compute Spearman rank correlations with cor(method='spearman') in R (vegan/tidyverse) between each tool's percent significant ASVs and each characteristic, separately for unfiltered and filtered data, then write the tool x characteristic coefficient table to spearman_correlations.csv.

## Related tools

- **vegan** (R package used for computing PERMANOVA effect sizes on Aitchison distance and supporting the correlation analysis) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **dplyr** (Used to compute the percent of significant ASVs per tool per dataset (version 0.8.5 listed in repo requirements)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **tidyverse** (Used for data assembly and Spearman correlation table construction) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
Rscript -e 'library(tidyverse); cor(pct_sig_asvs, effect_size, method="spearman")'
```

## Evaluation signals

- Unfiltered Spearman coefficients for percent significant ASVs vs effect size should fall within the reported 0.35–0.72 range
- Filtered (10% prevalence) Spearman coefficients should fall within the reported 0.31–0.56 range
- The reproduced relationships should match Fig. 2 of the manuscript
- spearman_correlations.csv must contain one coefficient per tool x characteristic combination, separately for unfiltered and filtered data
- The dataset characteristic table must include PERMANOVA effect sizes matching aitchison_permanova_results.tsv

## Limitations

- Whether rarefying is advisable for other analyses (e.g., prior to computing diversity metrics) is beyond the scope of this work
- The 10% prevalence filter cut-off was arbitrarily selected; more work is needed to estimate an optimal cut-off
- Not all environments and datasets are appropriate for cross-data consistency comparison, so correlations may not generalize across environments

## Evidence

- [abstract] For many tools the number of features identified correlates with sample size, sequencing depth, and effect size of community differences: "the number of features identified correlate with aspects of the data, such as sample size, sequencing depth, and effect size of community differences"
- [readme] run_no_filt_datasets.sh --> runs all tools on the 38 unfiltered datasets: "run_no_filt_datasets.sh --> runs all tools on the 38 unfiltered datasets"
- [readme] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [readme] Results of PERMANOVA tests based on Aitchison distance are stored in aitchison_permanova_results.tsv: [evidence span withheld — anchor status: not_found]
- [discussion] The paper recommends prevalence filtering at 10% prior to DA testing: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing"
