---
name: metabarcoding-false-positive-diagnosis
description: Use when metabarcoding ASV tables from different filtering pipelines
  (e.g. VTAM vs DALU/OBIbaR) are available and you suspect unfiltered false positives
  (tag-jumps, contaminants) inflating per-sample richness or deflating inter-sample
  dissimilarity.
when_to_use_negative:
- Only one pipeline's ASV table exists — the comparison-based diagnosis requires matched
  tables from at least two pipelines (e.g. VTAM vs DALU/OBIbaR) over the same samples
- The goal is to run or configure VTAM's own filtering steps (FilterLNF, FilterMinReplicateNumber,
  etc.) rather than to diagnose false positives in already-filtered output tables
- Input is raw FASTQ data rather than per-pipeline final taxa tables — pipeline processing
  must precede this diversity-based diagnosis
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: R (vegan package)
  role: Compute Jaccard distances (beta-diversity) between samples and run t-tests
    comparing ASV richness across pipelines
- name: VTAM
  role: Control-based variant filtering pipeline whose output serves as the high-precision
    reference against which DALU/OBIbaR richness is compared
  repo: https://github.com/aitgon/vtam
- name: vtam_benchmark
  role: Snakemake/singularity benchmark repository that produces the out/summary_min_readcout_{0,10,40,60}/
    final taxa tables consumed by this diagnosis
  repo: https://github.com/aitgon/vtam_benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:7ed210b1482f0312bd4fa414f2315317df9a8cdd199fbe11058fd01ca7b90472
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
  - R (vegan package)
  - VTAM
  - vtam_benchmark
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# metabarcoding-false-positive-diagnosis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Diagnoses metabarcoding false positives by comparing per-sample ASV richness (alpha-diversity) and Jaccard-based beta-diversity across filtering pipelines (VTAM, DALU, OBIbaR), then tracing inflated richness in suspect samples (e.g. unfiltered tag-jumps in five marine fish samples) back to their origin.

## When to use

Use when metabarcoding ASV tables from different filtering pipelines (e.g. VTAM vs DALU/OBIbaR) are available and you suspect unfiltered false positives (tag-jumps, contaminants) inflating per-sample richness or deflating inter-sample dissimilarity.

## When NOT to use

- Only one pipeline's ASV table exists — the comparison-based diagnosis requires matched tables from at least two pipelines (e.g. VTAM vs DALU/OBIbaR) over the same samples
- The goal is to run or configure VTAM's own filtering steps (FilterLNF, FilterMinReplicateNumber, etc.) rather than to diagnose false positives in already-filtered output tables
- Input is raw FASTQ data rather than per-pipeline final taxa tables — pipeline processing must precede this diversity-based diagnosis

## Inputs

- Nine per-pipeline final taxa tables (vtam/dalu/obibar × bat/fish/shark), e.g. dalu_bat_final_taxa.tsv from out/summary_min_readcout_{0,10,40,60}/
- Minimum read count cut-off values 0, 10, 40, 60
- Identification of suspect samples (e.g. the five marine samples from the fish dataset)

## Outputs

- alpha_beta_diversity_results.tsv with per-sample ASV richness, Jaccard distances, and t-test results per pipeline/dataset/cut-off
- Boxplots of diversity estimates per pipeline and cut-off (following bxplt_diversity_intraspecies.R logic)
- Confirmation (or refutation) of tag-jump false positives in reviewed samples

## How to apply

Load the per-pipeline final taxa tables (e.g. dalu_bat_final_taxa.tsv) from out/summary_min_readcout_{0,10,40,60}/ for the vtam/dalu/obibar × bat/fish/shark combinations. Compute per-sample ASV richness for each pipeline/dataset combination and Jaccard distances between samples using the vegan package in R. Compare alpha- and beta-diversity between the three pipelines with t-tests in R at minimum read count cut-offs 0, 10, 40, and 60 (VTAM-optimized cut-offs were 10 for fish, 40 for shark, and 60 for bat). Write the diversity estimates and t-test results to alpha_beta_diversity_results.tsv and produce boxplots following the logic of the bxplt_diversity_intraspecies.R script. Rationale: denoising/clustering algorithms filter PCR and sequencing errors but cannot account for inter-sample contamination, tag-jump, and chimeras, so pipelines lacking control-based filtering (DALU, OBIbaR) show spuriously higher richness; review ASVs in suspect samples — here the five marine samples of the fish dataset — to confirm tag-jump false positives as the cause.

## Related tools

- **R (vegan package)** (Compute Jaccard distances (beta-diversity) between samples and run t-tests comparing ASV richness across pipelines)
- **VTAM** (Control-based variant filtering pipeline whose output serves as the high-precision reference against which DALU/OBIbaR richness is compared) — https://github.com/aitgon/vtam
- **vtam_benchmark** (Snakemake/singularity benchmark repository that produces the out/summary_min_readcout_{0,10,40,60}/ final taxa tables consumed by this diagnosis) — https://github.com/aitgon/vtam_benchmark

## Examples

```
Rscript bxplt_diversity_intraspecies.R
```

## Evaluation signals

- T-tests must show significantly higher ASV richness in DALU and OBIbaR relative to VTAM — as reported for the bat dataset at all read count cut-offs (0, 10, 40, 60) and for the fish dataset at the 0 and 10 cut-offs
- Jaccard-based inter-sample dissimilarity must be correspondingly lower for DALU and OBIbaR than for VTAM
- Review of the five marine fish samples must confirm that unfiltered tag-jump false positives in DALU/OBIbaR inflate richness and deflate inter-sample dissimilarity
- alpha_beta_diversity_results.tsv must contain diversity estimates and t-test results for all four cut-offs (0, 10, 40, 60) across the nine pipeline/dataset combinations
- Boxplots must visually separate pipeline-specific richness distributions in the same direction as the t-test results

## Limitations

- Denoising/clustering algorithms alone filter PCR and sequencing errors but cannot account for inter-sample contamination, tag-jump, and chimeras — diversity inflation may reflect these unfiltered FPs rather than true biological richness
- The richness gap is cut-off dependent: DALU/OBIbaR precision increases with the minimum read count cut-off, and the fish-dataset significance disappears at higher cut-offs (40, 60)
- Shark dataset behaviour differs (significance at 0, 10, 40 cut-offs), so results do not transfer uniformly across datasets/markers (COI vs 16S)

## Evidence
