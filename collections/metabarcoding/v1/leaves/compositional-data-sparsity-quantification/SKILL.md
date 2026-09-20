---
name: compositional-data-sparsity-quantification
description: Use when you need to test whether differential abundance tool outputs
  scale with dataset characteristics such as effect size, sample size, sequencing
  depth, or sparsity, or to assess how prevalence filtering changes those relationships.
when_to_use_negative:
- You need a direct recommendation of a single best DA tool rather than a characterization
  of how significance counts scale with data characteristics
- Your datasets have no two-group comparison, so no PERMANOVA effect size on Aitchison
  distance can be computed
- You want absolute false positive rates rather than correlations with dataset properties
  — use the false positive analysis scripts (run_all_FP_datasets_*.sh) instead
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: vegan
  role: Source of Aitchison-distance-based PERMANOVA effect sizes used as the dataset
    characteristic
- name: dplyr
  role: Computes percent significant ASVs per tool per dataset
- name: tidyverse
  role: Data wrangling and assembly of the tool x characteristic correlation table
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:94349be321ffbf5ee137cf68ac20edb15e224bef2a16235ae5600e6ec78829e4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - dplyr
  - tidyverse
  - vegan
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# compositional-data-sparsity-quantification

## Summary

Quantifies whether the percent of significant ASVs identified by each of 14 differential abundance tools correlates with dataset characteristics (PERMANOVA effect size on Aitchison distance, sample size, read depth, richness, sparsity), separately for 38 unfiltered and 10%-prevalence-filtered 16S rRNA gene datasets.

## When to use

Use when you need to test whether differential abundance tool outputs scale with dataset characteristics such as effect size, sample size, sequencing depth, or sparsity, or to assess how prevalence filtering changes those relationships.

## When NOT to use

- You need a direct recommendation of a single best DA tool rather than a characterization of how significance counts scale with data characteristics
- Your datasets have no two-group comparison, so no PERMANOVA effect size on Aitchison distance can be computed
- You want absolute false positive rates rather than correlations with dataset properties — use the false positive analysis scripts (run_all_FP_datasets_*.sh) instead

## Inputs

- Per-tool significant ASV result files for 38 unfiltered datasets (run_no_filt_datasets.sh output)
- Per-tool significant ASV result files for 38 filtered datasets (run_0.1_filt_datasets.sh output, -F 0.1)
- aitchison_permanova_results.tsv (PERMANOVA effect sizes on Aitchison distance)
- ASV count table and metadata (mapfile) per dataset for sample size, read depth, richness, and sparsity

## Outputs

- spearman_correlations.csv: tool x characteristic Spearman rho table for unfiltered and filtered conditions
- Per-dataset percent significant ASV table per tool (dplyr summary)

## How to apply

Load per-tool significant ASV results for the 38 unfiltered datasets (from run_no_filt_datasets.sh) and the 38 filtered datasets (from run_0.1_filt_datasets.sh with -F 0.1, i.e. a 10% prevalence filter). Use dplyr to compute, per dataset and per tool, the percent of significant ASVs relative to the total ASVs tested. Assemble a dataset characteristic table with PERMANOVA effect size on Aitchison distance from aitchison_permanova_results.tsv plus sample size, median/range read depth, richness, and sparsity. Compute Spearman rank correlations with cor(method='spearman') in R (vegan/tidyverse) between each tool's percent significant ASVs and each characteristic, separately for unfiltered and filtered data, then assemble the tool x characteristic coefficient table (spearman_correlations.csv) for both conditions.

## Related tools

- **vegan** (Source of Aitchison-distance-based PERMANOVA effect sizes used as the dataset characteristic)
- **dplyr** (Computes percent significant ASVs per tool per dataset)
- **tidyverse** (Data wrangling and assembly of the tool x characteristic correlation table)

## Examples

```
Rscript -e 'df <- read.csv("spearman_correlations.csv"); rho <- cor(df$pct_sig, df$effect_size, method="spearman")'
```

## Evaluation signals

- Unfiltered Spearman coefficients fall within the reported 0.35–0.72 range
- Filtered (10% prevalence) Spearman coefficients fall within the reported 0.31–0.56 range
- Relationships reproduce Fig. 2 of the manuscript (number of significant ASVs vs effect size, sample size, sequencing depth)
- spearman_correlations.csv contains one rho value per tool per characteristic for BOTH filtering conditions
- Effect size column matches values in aitchison_permanova_results.tsv from Misc_datafiles

## Limitations

- Correlations are rank-based (Spearman), so they capture only monotonic relationships and are sensitive to ties in small-n settings
- Percent significant ASVs depend on the total ASV count, which changes substantially under the 10% prevalence filter, so unfiltered and filtered coefficients are not directly comparable
- PERMANOVA effect size on Aitchison distance is a single community-level statistic and may not capture feature-level effects driving per-tool significance counts

## Evidence

- [abstract] the number of features identified correlate with aspects of the data, such as sample size, sequencing depth, and effect size of community differences: "the number of features identified correlate with aspects of the data, such as sample size, sequencing depth, and effect size of community differences"
- [methods] run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [methods] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
- [readme] Results of PERMANOVA tests based on Aitchison distance (_aitchison_permanova_results.tsv_): [evidence span withheld — anchor status: not_found]
- [readme] The key script is "run_all_tools.sh" for running all tools on a given dataset.: "The key script is "run_all_tools.sh" for running all tools on a given dataset."
