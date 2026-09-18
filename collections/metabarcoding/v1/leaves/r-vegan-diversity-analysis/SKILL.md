---
name: r-vegan-diversity-analysis
description: Use when comparing per-sample ASV richness and inter-sample Jaccard dissimilarity
  across multiple metabarcoding filtering pipelines on the same biological samples,
  e.g. VTAM vs DALU vs OBIbaR taxa tables at several read count cut-offs.
when_to_use_negative:
- Input is still raw FASTQ or an unfiltered ASV table — this skill expects pipeline-final
  taxa tables (e.g. *_final_taxa.tsv) produced after VTAM/DALU/OBIbaR filtering
- The goal is taxonomy-free denoising or clustering of reads (Swarm, Unoise, DADA2)
  rather than post-hoc diversity comparison between already-run pipelines
- Comparing a single pipeline against itself with no cross-pipeline contrast — the
  t-test framework is built for VTAM vs DALU vs OBIbaR comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: vegan
  role: R package used to compute Jaccard distances between samples (beta-diversity)
    from the taxa tables
- name: R
  role: Language used to compute per-sample ASV richness, run t-tests between pipelines,
    and generate boxplots
- name: VTAM
  role: Source of one of the three compared final taxa tables; its stringent control-based
    filtering is the reference pipeline against which DALU and OBIbaR richness is
    tested
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:49bf278eac3f0f88ec2d85d490ffc98b4bdece2f89af4ba355b5ca474e45d547
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
  - dadasnake
  - vegan
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# r-vegan-diversity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

R/vegan-based comparison of alpha-diversity (per-sample ASV richness) and beta-diversity (Jaccard distance) between metabarcoding pipelines (VTAM, DALU, OBIbaR) across the bat, fish, and shark datasets, tested with t-tests over minimum read count cut-offs 0, 10, 40, and 60.

## When to use

Use when comparing per-sample ASV richness and inter-sample Jaccard dissimilarity across multiple metabarcoding filtering pipelines on the same biological samples, e.g. VTAM vs DALU vs OBIbaR taxa tables at several read count cut-offs.

## When NOT to use

- Input is still raw FASTQ or an unfiltered ASV table — this skill expects pipeline-final taxa tables (e.g. *_final_taxa.tsv) produced after VTAM/DALU/OBIbaR filtering
- The goal is taxonomy-free denoising or clustering of reads (Swarm, Unoise, DADA2) rather than post-hoc diversity comparison between already-run pipelines
- Comparing a single pipeline against itself with no cross-pipeline contrast — the t-test framework is built for VTAM vs DALU vs OBIbaR comparison

## Inputs

- Nine per-pipeline final taxa tables (vtam/dalu/obibar x bat/fish/shark), e.g. dalu_bat_final_taxa.tsv from out/summary_min_readcout_{0,10,40,60}/
- Minimum read count cut-off values: 0, 10, 40, 60
- Five marine fish samples from the fish dataset (tag-jump diagnostic subset)

## Outputs

- Per-sample ASV richness (alpha-diversity) estimates per pipeline/dataset/cut-off combination
- Jaccard-based inter-sample dissimilarity (beta-diversity) matrices per pipeline/dataset/cut-off combination
- T-test comparison results across VTAM, DALU, and OBIbaR
- alpha_beta_diversity_results.tsv
- Boxplots of diversity estimates following bxplt_diversity_intraspecies.R logic

## How to apply

Load the nine per-pipeline final taxa tables (vtam/dalu/obibar x bat/fish/shark) from out/summary_min_readcout_{0,10,40,60}/, e.g. dalu_bat_final_taxa.tsv, and compute per-sample ASV richness for each pipeline/dataset combination. Compute pairwise Jaccard distances between samples with the vegan package in R (e.g. vegdist(..., method='jaccard')). Compare richness and dissimilarity between the three pipelines with t-tests in R, repeating the comparison across the minimum read count cut-offs 0, 10, 40, and 60. Examine the five marine samples of the fish dataset to confirm that unfiltered tag-jump false positives in DALU/OBIbaR inflate richness and deflate inter-sample dissimilarity. Write the diversity estimates and t-test results to alpha_beta_diversity_results.tsv and produce boxplots following the logic of the bxplt_diversity_intraspecies.R script.

## Related tools

- **vegan** (R package used to compute Jaccard distances between samples (beta-diversity) from the taxa tables)
- **R** (Language used to compute per-sample ASV richness, run t-tests between pipelines, and generate boxplots)
- **VTAM** (Source of one of the three compared final taxa tables; its stringent control-based filtering is the reference pipeline against which DALU and OBIbaR richness is tested) — https://github.com/aitgon/vtam

## Examples

```
Rscript -e 'library(vegan); jac_dist <- vegdist(t(final_taxa_table), method="jaccard")'
```

## Evaluation signals

- T-tests show significantly higher ASV richness in DALU and OBIbaR relative to VTAM: bat dataset at all read count cut-offs (0, 10, 40, 60) and fish dataset at cut-offs 0 and 10
- Jaccard-based inter-sample dissimilarity is correspondingly lower for DALU and OBIbaR than for VTAM at the same cut-offs
- Reviewing the five marine fish samples confirms unfiltered tag-jump false positives in DALU/OBIbaR inflate richness and deflate dissimilarity
- alpha_beta_diversity_results.tsv is populated with diversity estimates and t-test results for all nine pipeline/dataset combinations across all four cut-offs
- Boxplots reproduce the layout/logic of the bxplt_diversity_intraspecies.R script

## Limitations

- Unfiltered tag-jump false positives in DALU/OBIbaR inflate ASV richness and deflate inter-sample dissimilarity, so alpha/beta-diversity differences alone can reflect false positives rather than true diversity
- Significant richness differences between pipelines are dataset- and cut-off-dependent: they hold for the bat dataset at all cut-offs and the fish dataset only at 0 and 10 cut-offs (and shark at 0, 10, 40), so results at other cut-off/dataset combinations may not replicate
- VTAM's suggested read-count cut-offs differ per dataset (10 for fish, 40 for shark, 60 for bat), complicating like-for-like cut-off comparisons

## Evidence
