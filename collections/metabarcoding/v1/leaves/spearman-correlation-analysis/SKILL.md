---
name: spearman-correlation-analysis
description: Use when you have per-tool significant-ASV counts across many datasets
  plus a table of dataset characteristics and need to test whether DA output volume
  tracks effect size, sample size, sequencing depth, or sparsity.
when_to_use_negative:
- You need correlations of raw significant-ASV counts rather than percent of significant
  ASVs per dataset — the analysis uses per-dataset percentages computed with dplyr,
  not absolute counts
- You need parametric significance testing of tool-vs-characteristic relationships
  (e.g., linear models with p-values) rather than non-parametric rank correlation
  coefficients
- Your dataset has fewer than ~38 datasets, making the tool x characteristic correlation
  table unstable and outside the validated 0.35–0.72 / 0.31–0.56 ranges
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: vegan
  role: R package used for computing Spearman rank correlations between percent significant
    ASVs and dataset characteristics
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: dplyr
  role: R package (v0.8.5) used to compute the percent of significant ASVs per tool
    per dataset before correlation
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- name: tidyverse
  role: R framework used for data assembly and correlation table manipulation
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:8612fb1152a78418c726549d8b804167b212092fa129f0194e15b6631d576867
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
  tools:
  - dplyr
  - tidyverse
  - vegan
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# spearman-correlation-analysis

## Summary

Computes Spearman rank correlations between the percent of significant ASVs per differential abundance tool and dataset characteristics (PERMANOVA effect size on Aitchison distance, sample size, read depth, richness, sparsity), separately for unfiltered and 10% prevalence-filtered 16S rRNA datasets, to quantify how DA tool output scales with data properties.

## When to use

Use when you have per-tool significant-ASV counts across many datasets plus a table of dataset characteristics and need to test whether DA output volume tracks effect size, sample size, sequencing depth, or sparsity.

## When NOT to use

- You need correlations of raw significant-ASV counts rather than percent of significant ASVs per dataset — the analysis uses per-dataset percentages computed with dplyr, not absolute counts
- You need parametric significance testing of tool-vs-characteristic relationships (e.g., linear models with p-values) rather than non-parametric rank correlation coefficients
- Your dataset has fewer than ~38 datasets, making the tool x characteristic correlation table unstable and outside the validated 0.35–0.72 / 0.31–0.56 ranges

## Inputs

- Per-tool significant ASV results for the 38 unfiltered datasets (output of run_no_filt_datasets.sh)
- Per-tool significant ASV results for the 38 filtered datasets (output of run_0.1_filt_datasets.sh with -F 0.1 prevalence filter)
- aitchison_permanova_results.tsv (PERMANOVA effect sizes on Aitchison distance, from Misc_datafiles)
- Dataset characteristics table: sample size, median/range read depth, richness, and sparsity per dataset

## Outputs

- spearman_correlations.csv: tool x characteristic Spearman correlation coefficients for both unfiltered and filtered conditions
- Reproduction of the Fig. 2 correlation relationships

## How to apply

Load the per-tool significant ASV results for the 38 unfiltered datasets (produced by run_no_filt_datasets.sh) and the 38 filtered datasets (produced by run_0.1_filt_datasets.sh with a 10% prevalence filter, -F 0.1). For each dataset compute the percent of significant ASVs per tool with dplyr. Assemble the dataset characteristics table using PERMANOVA effect size on Aitchison distance from aitchison_permanova_results.tsv (provided in Misc_datafiles of the repo), plus sample size, median/range read depth, richness, and sparsity per dataset. Compute Spearman rank correlations with cor(method='spearman') in R (vegan/tidyverse) between each tool's percent significant ASVs and each characteristic, separately for unfiltered and filtered data. Assemble the tool x characteristic correlation coefficients into a single table for both filtering conditions and write it to spearman_correlations.csv. Validation checks that unfiltered coefficients fall in the reported 0.35–0.72 range and filtered coefficients in the 0.31–0.56 range, reproducing the relationships shown in Fig. 2.

## Related tools

- **vegan** (R package used for computing Spearman rank correlations between percent significant ASVs and dataset characteristics) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **dplyr** (R package (v0.8.5) used to compute the percent of significant ASVs per tool per dataset before correlation) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods
- **tidyverse** (R framework used for data assembly and correlation table manipulation) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
cor(x = pct_sig_asvs[[tool]], y = dataset_chars$aitchison_effect_size, method = "spearman", use = "complete.obs")
```

## Evaluation signals

- Unfiltered Spearman coefficients for percent significant ASVs vs effect size fall within the reported range 0.35–0.72 across tools
- Filtered (10% prevalence) Spearman coefficients fall within the reported range 0.31–0.56 across tools
- The tool x characteristic correlation table reproduces the relationships shown in Fig. 2 of the manuscript
- spearman_correlations.csv contains one row per DA tool with columns for PERMANOVA Aitchison effect size, sample size, read depth, richness, and sparsity under both filtering conditions
- Percent significant ASV values per tool per dataset are between 0 and 100 and computed against the same dataset set (38 unfiltered, 38 filtered)

## Limitations

- Spearman correlations capture monotonic association only; they do not establish why some methods have much higher significance rates than others (a question the authors state is unclear)
- The 10% prevalence filter cut-off was arbitrarily selected; an optimal cut-off is unknown, so filtered-vs-unfiltered coefficient shifts should be interpreted cautiously
- Percent significant ASV values depend on the upstream DA tool versions (e.g., ALDEx2 v1.18.0, DESeq2 v1.26.0, R 3.6.3) and the docker image dockerdkd/hackathon2021; different tool versions may shift coefficients outside the validated ranges

## Evidence

- [results] The number of ASVs identified by all tools positively correlated with the effect size between test groups (Spearman's rho 0.35–0.72 with unfiltered data and 0.31–0.56 for filtered data).: [evidence span withheld — anchor status: not_found]
- [abstract] For many tools the number of features identified correlates with sample size, sequencing depth, and effect size of community differences: "the number of features identified correlate with aspects of the data, such as sample size, sequencing depth, and effect size of community differences"
- [readme] Results of PERMANOVA tests based on Aitchison distance (_aitchison_permanova_results.tsv_): [evidence span withheld — anchor status: not_found]
- [readme] run_no_filt_datasets.sh --> runs all tools on the 38 unfiltered datasets: "run_0.1_filt_datasets.sh --> runs all tools on the 38 filtered datasets"
- [methods] -F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter: "-F represents the prevelance filtering. I.e. 0.1 represents a 10% prevelance filter (the used within the analysis of the linked manuscript)"
