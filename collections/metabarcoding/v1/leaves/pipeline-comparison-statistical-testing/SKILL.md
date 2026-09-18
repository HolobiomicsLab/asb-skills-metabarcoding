---
name: pipeline-comparison-statistical-testing
description: Use when you have final ASV/taxa tables produced by two or more metabarcoding
  pipelines over the same samples and need to test whether pipelines differ in alpha-diversity
  (richness) or beta-diversity (inter-sample dissimilarity).
when_to_use_negative:
- Input is raw FASTQ data — this skill operates on already-filtered final ASV/taxa
  tables, not on reads.
- The goal is to measure sensitivity or precision of the pipelines (per-variant FP/FN
  against mock/replicate controls) rather than diversity-based comparison.
- The taxa tables come from a single pipeline — at least two pipelines over the same
  samples are needed for the t-test comparison.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0610
tools:
- name: R
  role: Statistical environment for computing ASV richness, Jaccard beta-diversity
    and t-tests; boxplots follow bxplt_diversity_intraspecies.R
- name: vegan
  role: R package used to compute Jaccard distances between samples (beta-diversity)
- name: VTAM
  role: One of the three compared pipelines; its filter outputs and taxassign command
    provide the final taxa tables benchmarked here
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:792d84171e87a1f29bfc162c70cd28657a0c999e6241952ad3c458caba5bd447
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aitgon/vtam_benchmark
  repo_url: https://github.com/aitgon/vtam
  tools:
  - R
  - VTAM
  - vegan
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# pipeline-comparison-statistical-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduces the benchmark logic of comparing metabarcoding pipelines (VTAM vs DALU vs OBIbaR) by computing per-sample ASV richness and Jaccard-based beta-diversity from per-pipeline final taxa tables and testing differences with t-tests in R across minimum read count cut-offs 0, 10, 40, and 60.

## When to use

Use when you have final ASV/taxa tables produced by two or more metabarcoding pipelines over the same samples and need to test whether pipelines differ in alpha-diversity (richness) or beta-diversity (inter-sample dissimilarity).

## When NOT to use

- Input is raw FASTQ data — this skill operates on already-filtered final ASV/taxa tables, not on reads.
- The goal is to measure sensitivity or precision of the pipelines (per-variant FP/FN against mock/replicate controls) rather than diversity-based comparison.
- The taxa tables come from a single pipeline — at least two pipelines over the same samples are needed for the t-test comparison.

## Inputs

- Per-pipeline final taxa tables, e.g. dalu_bat_final_taxa.tsv, from out/summary_min_readcout_{0,10,40,60}/ (vtam/dalu/obibar × bat/fish/shark)
- Minimum read count cut-off values 0, 10, 40, 60
- Sample metadata distinguishing the five marine fish samples in the fish dataset (for tag-jump inspection)

## Outputs

- alpha_beta_diversity_results.tsv (per-sample ASV richness and Jaccard dissimilarity estimates plus t-test results per pipeline/dataset/cut-off)
- Boxplots of alpha- and beta-diversity per pipeline following bxplt_diversity_intraspecies.R

## How to apply

Load the nine per-pipeline final taxa tables (vtam/dalu/obibar × bat/fish/shark) from out/summary_min_readcout_{0,10,40,60}/, e.g. dalu_bat_final_taxa.tsv. Compute per-sample ASV richness for each pipeline/dataset combination and Jaccard distances between samples using the vegan package in R, then compare pipelines pairwise with t-tests at each of the four read count cut-offs (0, 10, 40, 60). Expected direction of results: DALU and OBIbaR show higher richness and lower inter-sample dissimilarity than VTAM, because unfiltered tag-jump false positives inflate richness — confirm this by examining the five marine fish samples of the fish dataset. Write estimates and t-test results to alpha_beta_diversity_results.tsv and produce boxplots following the logic of the bxplt_diversity_intraspecies.R script. Taxonomic assignment homogeneity matters: use the same assignment step across pipelines (VTAM's taxassign command was used for all three pipelines) so diversity differences are not confounded by differing taxonomic backends.

## Related tools

- **R** (Statistical environment for computing ASV richness, Jaccard beta-diversity and t-tests; boxplots follow bxplt_diversity_intraspecies.R)
- **vegan** (R package used to compute Jaccard distances between samples (beta-diversity))
- **VTAM** (One of the three compared pipelines; its filter outputs and taxassign command provide the final taxa tables benchmarked here) — https://github.com/aitgon/vtam

## Examples

```
Rscript bxplt_diversity_intraspecies.R
```

## Evaluation signals

- t-tests show significantly higher ASV richness in DALU and OBIbaR relative to VTAM for the bat dataset at all read count cut-offs and for the fish dataset at the 0 and 10 cut-offs
- Jaccard-based inter-sample dissimilarity is correspondingly lower for DALU and OBIbaR than for VTAM
- The five marine fish samples of the fish dataset show tag-jump false positives in DALU/OBIbaR that inflate richness and deflate inter-sample dissimilarity
- All nine pipeline/dataset combinations × four cut-offs are represented in alpha_beta_diversity_results.tsv
- Taxonomic assignment is homogeneous across pipelines (all assigned with VTAM's taxassign command) before diversity comparison

## Limitations

- DALU and OBIbaR pipelines are referenced by name only in the article, without tool identity, version, or hosting location
- No accession or identifier is given for the three datasets (bat, fish, shark) beyond Dryad/OSF links in the benchmark repo
- Differences in richness can be driven by unfiltered tag-jump false positives in DALU/OBIbaR rather than genuine biological signal, so findings must be interpreted alongside precision results

## Evidence
