---
name: jaccard-beta-diversity-estimation
description: Use when comparing per-sample ASV richness and inter-sample Jaccard dissimilarity
  between metabarcoding pipelines, or when assessing whether tag-jump false positives
  inflate diversity estimates in amplicon taxa tables.
when_to_use_negative:
- Input is already a per-sample richness or dissimilarity summary rather than a sample×ASV
  final taxa table.
- The analysis goal is sensitivity/precision benchmarking against control samples
  (mocks/negatives) rather than between-pipeline diversity comparison.
- The taxa table has no sample-level information (e.g. only a variant list), since
  neither richness per sample nor Jaccard distances can be computed.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: vegan
  role: R package used to compute Jaccard distances between samples (beta-diversity)
- name: R
  role: Environment for computing richness, Jaccard dissimilarity, t-tests, and boxplots
- name: VTAM
  role: One of the three pipelines whose final taxa tables supply occurrence data
    for the diversity comparison
  repo: https://github.com/aitgon/vtam
- name: vtam_benchmark
  role: Repository producing the per-pipeline final taxa tables (out/summary_min_readcout_{0,10,40,60}/)
    used as diversity-comparison inputs
  repo: https://github.com/aitgon/vtam_benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:19163091b9ebb7a054d9fa8475e20043c8d13d775e865f96b412b16051fac135
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
  - vtam_benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# jaccard-beta-diversity-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-sample ASV richness (alpha-diversity) and Jaccard-based inter-sample dissimilarity (beta-diversity) from per-pipeline final taxa tables to compare metabarcoding pipelines (VTAM vs DALU vs OBIbaR) across minimum read count cut-offs.

## When to use

Use when comparing per-sample ASV richness and inter-sample Jaccard dissimilarity between metabarcoding pipelines, or when assessing whether tag-jump false positives inflate diversity estimates in amplicon taxa tables.

## When NOT to use

- Input is already a per-sample richness or dissimilarity summary rather than a sample×ASV final taxa table.
- The analysis goal is sensitivity/precision benchmarking against control samples (mocks/negatives) rather than between-pipeline diversity comparison.
- The taxa table has no sample-level information (e.g. only a variant list), since neither richness per sample nor Jaccard distances can be computed.

## Inputs

- nine per-pipeline final taxa tables (vtam/dalu/obibar × bat/fish/shark) from out/summary_min_readcout_{0,10,40,60}/, e.g. dalu_bat_final_taxa.tsv
- minimum read count cut-off values 0, 10, 40, and 60

## Outputs

- per-sample ASV richness (alpha-diversity) estimates per pipeline/dataset/cut-off
- Jaccard inter-sample dissimilarity (beta-diversity) estimates per pipeline/dataset/cut-off
- t-test comparison results written to alpha_beta_diversity_results.tsv
- diversity boxplots (following bxplt_diversity_intraspecies.R logic)

## How to apply

Load the nine per-pipeline final taxa tables (vtam/dalu/obibar × bat/fish/shark) from out/summary_min_readcout_{0,10,40,60}/ (e.g. dalu_bat_final_taxa.tsv) and compute per-sample ASV richness for each pipeline/dataset combination. In R, compute Jaccard distances between samples with the vegan package, then compare alpha- and beta-diversity between the three pipelines with t-tests across minimum read count cut-offs 0, 10, 40, and 60. Inspect the five marine fish samples of the fish dataset specifically, because unfiltered tag-jump false positives in DALU/OBIbaR inflate richness and deflate inter-sample dissimilarity there. Write estimates and t-test results to alpha_beta_diversity_results.tsv and produce boxplots following the logic of the bxplt_diversity_intraspecies.R script. Validation: the t-tests must show significantly higher ASV richness in DALU and OBIbaR relative to VTAM (bat dataset at all cut-offs; fish dataset at 0 and 10 cut-offs; shark dataset at 0, 10, and 40 cut-offs), with correspondingly lower Jaccard-based inter-sample dissimilarity for DALU and OBIbaR.

## Related tools

- **vegan** (R package used to compute Jaccard distances between samples (beta-diversity))
- **R** (Environment for computing richness, Jaccard dissimilarity, t-tests, and boxplots)
- **VTAM** (One of the three pipelines whose final taxa tables supply occurrence data for the diversity comparison) — https://github.com/aitgon/vtam
- **vtam_benchmark** (Repository producing the per-pipeline final taxa tables (out/summary_min_readcout_{0,10,40,60}/) used as diversity-comparison inputs) — https://github.com/aitgon/vtam_benchmark

## Examples

```
Rscript -e 'library(vegan); D <- vegdist(asv_table, method="jaccard")'
```

## Evaluation signals

- t-tests show significantly higher ASV richness in DALU and OBIbaR compared to VTAM: bat dataset for all read count cut-offs, fish dataset with 0 and 10 cut-offs, shark dataset with 0, 10, and 40 cut-offs
- Jaccard-based inter-sample dissimilarity is correspondingly lower for DALU and OBIbaR than for VTAM
- Review of the five marine fish samples confirms unfiltered tag-jump false positives inflate DALU/OBIbaR richness and deflate inter-sample dissimilarity
- Results file alpha_beta_diversity_results.tsv exists with diversity estimates and t-test results across cut-offs 0, 10, 40, 60
- Boxplots are generated consistent with the logic of bxplt_diversity_intraspecies.R

## Limitations

- Unfiltered tag-jump false positives in DALU/OBIbaR inflate per-sample richness and deflate inter-sample dissimilarity, most visibly in the five marine fish samples
- Divergence in diversity outcomes depends on the minimum read count cut-off (0, 10, 40, 60); e.g. richness differences in the fish dataset are significant only at 0 and 10 cut-offs
- VTAM suggested cut-offs differ per dataset (10 for fish, 40 for shark, 60 for bat), so cross-dataset comparison must account for dataset-specific optimized thresholds

## Evidence
