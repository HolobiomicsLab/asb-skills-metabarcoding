---
name: compositional-data-analysis
description: Reference for testing which taxa differ in relative abundance between
  sample groups when the input is a 16S rRNA ASV/OTU count table, since sequencing
  provides only relative abundance information.
when_to_use_negative:
- The abundance table is already transformed to relative abundance (column totals
  equal to 1) — ANCOM requires the absolute table
- The goal is a different analysis such as computing diversity metrics — the paper
  states whether rarefying is advisable for such analyses is beyond its scope
- A single tool's result will be reported without running several methods — the authors
  recommend a consensus approach based on multiple DA methods
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3053
tools:
- name: ANCOM / ANCOM-II
  role: Implements the additive log-ratio approach; feature_table_pre_process handles
    structural zeros, outlier zeros, and sampling zeros before DA testing
  repo: FrederickHuangLin/ANCOM-Code-Archive
- name: ANCOMBC (ancom / ancombc R functions)
  role: Recommended maintained replacement for the archived standalone ANCOM functions
  repo: https://www.bioconductor.org/packages/release/bioc/html/ANCOMBC.html
- name: ALDEx2
  role: Extension of the compositional log-ratio approach; among the most consistent
    tools across the 38 datasets
- name: run_all_tools.sh (Comparison_of_DA_microbiome_methods pipeline)
  role: Runs ANCOM and other DA tools uniformly on a dataset with prevalence filtering
    (-F 0.1) and rarefaction depth (-D 2000)
  repo: https://github.com/nearinj/Comparison_of_DA_microbiome_methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-28034-z
    title: Microbiome differential abundance methods produce different results across
      38 datasets
  source_content_hash: sha256:2e25e7d6ac377568879209efcb4c5d951a3533041c37acce115e9134db280198
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/FrederickHuangLin/ANCOM-Code-Archive
  tools:
  - ALDEx2
  - ANCOM / ANCOM-II
  - ANCOM-II
  - ANCOMBC (ancom / ancombc R functions)
  - DESeq2
  - LEfSe
  - Maaslin2
  - corncob
  - edgeR
  - limma voom
  - metagenomeSeq
  - run_all_tools.sh (Comparison_of_DA_microbiome_methods pipeline)
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# compositional-data-analysis

## Summary

Treat 16S rRNA read counts as compositional data (CoDa) rather than absolute abundances, using log-ratio based differential abundance tools (ALDEx2, ANCOM-II) that analyze ratios of read counts between taxa, with ANCOM-II-style zero handling and prevalence filtering before testing.

## When to use

Reference for testing which taxa differ in relative abundance between sample groups when the input is a 16S rRNA ASV/OTU count table, since sequencing provides only relative abundance information.

## When NOT to use

- The abundance table is already transformed to relative abundance (column totals equal to 1) — ANCOM requires the absolute table
- The goal is a different analysis such as computing diversity metrics — the paper states whether rarefying is advisable for such analyses is beyond its scope
- A single tool's result will be reported without running several methods — the authors recommend a consensus approach based on multiple DA methods

## Inputs

- Absolute abundance ASV/OTU table (taxa in rows, samples in columns; must NOT be pre-normalized to relative abundances)
- Metadata table with sample IDs and a group indicator (two-column TSV: sample name, grouping; maximum 2 groupings in the comparison pipeline)
- Optional covariates and formulas for adjustment (adj_formula, rand_formula) and lme_control for mixed models

## Outputs

- Pre-processed feature table and metadata from feature_table_pre_process
- Structural zeros matrix (0/1 per taxon per group)
- Per-taxon differential abundance results with BH-adjusted p-values at alpha 0.05
- Percentage of significant ASVs per tool per dataset (e.g., significant_asv_percentages.tsv when run across the 38 datasets)

## How to apply

Start by recognizing that sequencing data are compositional — only relative abundances are observable — so standard tests meant for absolute abundances can yield false inferences; use tools that analyze ratios of read counts between taxa, such as ANCOM (additive log-ratio approach) and ALDEx2 (an extension of this approach). For ANCOM, run feature_table_pre_process on the absolute abundance table (taxa in rows, samples in columns — do not convert to relative abundance) with parameters out_cut=0.05, zero_cut=0.90, a lib_cut threshold, and neg_lb to classify structural zeros per group. Apply prevalence filtering of features (e.g., at 10%, as in -F 0.1 of the comparison pipeline) prior to DA testing, acknowledging the cut-off was arbitrarily selected. Run the ANCOM main function with p_adj_method 'BH' and alpha 0.05, supplying group_var so structural zeros and outliers can be detected. Interpret results in light of the benchmark on 38 16S rRNA datasets, where ALDEx2 and ANCOM-II were the most consistent tools and best agreed with the intersect of different approaches (ANCOM-II filtered mean 3.8% significant ASVs, SD 5.9%).

## Related tools

- **ANCOM / ANCOM-II** (Implements the additive log-ratio approach; feature_table_pre_process handles structural zeros, outlier zeros, and sampling zeros before DA testing) — FrederickHuangLin/ANCOM-Code-Archive
- **ANCOMBC (ancom / ancombc R functions)** (Recommended maintained replacement for the archived standalone ANCOM functions) — https://www.bioconductor.org/packages/release/bioc/html/ANCOMBC.html
- **ALDEx2** (Extension of the compositional log-ratio approach; among the most consistent tools across the 38 datasets)
- **run_all_tools.sh (Comparison_of_DA_microbiome_methods pipeline)** (Runs ANCOM and other DA tools uniformly on a dataset with prevalence filtering (-F 0.1) and rarefaction depth (-D 2000)) — https://github.com/nearinj/Comparison_of_DA_microbiome_methods

## Examples

```
source("programs/ancom.R"); pre <- feature_table_pre_process(feature_table, meta_data, sample_var, group_var = "group", out_cut = 0.05, zero_cut = 0.90, neg_lb = TRUE)
```

## Evaluation signals

- Input check: the feature table passed to ANCOM must be absolute abundance; relative abundance tables (column totals = 1) are explicitly disallowed by the ANCOM README
- Zero handling check: taxa with a proportion of zeroes greater than zero_cut (0.90) must be excluded from the analysis; structural zeros must be flagged per group when neg_lb=TRUE
- Benchmark check: on 10% prevalence-filtered data ANCOM-II should be among the most stringent tools (mean 3.8% significant ASVs, SD 5.9%), far below liberal tools like edgeR (mean 32.5%)
- Pipeline check: running run_all_tools.sh on the Human-ALL (MALL) dataset with -F 0.1 -D 2000 should match Misc_datafiles/Expected_MALL_output in ~20 minutes on a single core
- Consistency check: ALDEx2 and ANCOM-II results should agree best with the intersect of results from different approaches

## Limitations

- The 10% prevalence filtering cut-off was arbitrarily selected; the optimal cut-off is unknown
- The randomized approach for estimating FDR is not a perfect representation of real data; few significant ASVs is not proof of low practical FDR
- There is no assurance that the intersection (consensus) of tool outputs is more reliable — tools may simply be picking up the same noise
- ANCOM v2.1's main_var must be categorical; the standalone archived ANCOM code is no longer maintained (use ANCOMBC)

## Evidence

- [intro] sequencing data are compositional, so standard methods intended for absolute abundances can produce false inferences when applied to relative abundances; CoDa methods instead analyze ratios of read counts between taxa: "it has recently become more widely appreciated that sequencing data are compositional12, meaning that sequencing only provides information on the relative abundance of features"
- [intro] ANCOM implements the additive log-ratio approach for compositional DA testing: "ANCOM is one tool that implements this additive log-ratio approach16"
- [readme] ALDEx2 must receive an absolute abundance table, not a relative one: [evidence span withheld — anchor status: not_found]
- [readme] ANCOM preprocessing follows ANCOM-II methodology for handling different types of zeros before DA analysis: [evidence span withheld — anchor status: not_found]
- [abstract] ALDEx2 and ANCOM-II are the most consistent DA tools across studies: "ALDEx2 and ANCOM-II produce the most consistent results across studies and agree best with the intersect of results from different approaches."
- [discussion] Prevalence filtering at ~10% of features is recommended prior to DA testing: "we would suggest performing prevalence filtering (e.g., at 10%) of features prior to DA testing, although we acknowledge that more work is needed to estimate an optimal cut-off rather than just"
