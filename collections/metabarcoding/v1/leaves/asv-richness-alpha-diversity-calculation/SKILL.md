---
name: asv-richness-alpha-diversity-calculation
description: Use when comparing amplicon/ASV tables produced by multiple metabarcoding
  pipelines (e.g., VTAM vs DALU vs OBIbaR) to quantify whether filtering stringency
  changes per-sample richness and inter-sample Jaccard dissimilarity.
when_to_use_negative:
- Input is already a diversity/t-test result table rather than a per-sample ASV count
  or final taxa table
- The comparison does not involve control-validated pipeline outputs (e.g., raw FASTQ
  without a per-pipeline ASV table from VTAM/DALU/OBIbaR filtering)
- Single-pipeline analyses with no alternative pipeline to contrast richness against
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VTAM
  role: Produces the control-validated final taxa tables whose ASV richness is compared;
    its filtering yields the VTAM baseline pipeline results
  repo: https://github.com/aitgon/vtam
- name: R
  role: Computes per-sample ASV richness, t-tests between pipelines, and diversity
    boxplots
- name: vegan
  role: R package used to compute Jaccard distances between samples (beta-diversity)
- name: vtam_benchmark
  role: Snakemake/Singularity repository reproducing the benchmark that generated
    the per-pipeline output tables
  repo: https://github.com/aitgon/vtam_benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:432c40889bd999256bbe79837708819d0f94749424d8c5553a23ecde340ca97e
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
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# asv-richness-alpha-diversity-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-sample ASV richness (alpha-diversity) and Jaccard-based beta-diversity from per-pipeline final taxa tables (VTAM, DALU, OBIbaR) to benchmark metabarcoding pipelines across read-count cut-offs using t-tests in R.

## When to use

Use when comparing amplicon/ASV tables produced by multiple metabarcoding pipelines (e.g., VTAM vs DALU vs OBIbaR) to quantify whether filtering stringency changes per-sample richness and inter-sample Jaccard dissimilarity.

## When NOT to use

- Input is already a diversity/t-test result table rather than a per-sample ASV count or final taxa table
- The comparison does not involve control-validated pipeline outputs (e.g., raw FASTQ without a per-pipeline ASV table from VTAM/DALU/OBIbaR filtering)
- Single-pipeline analyses with no alternative pipeline to contrast richness against

## Inputs

- Per-pipeline final taxa tables (TSV, e.g. dalu_bat_final_taxa.tsv) from out/summary_min_readcout_{0,10,40,60}/ for vtam/dalu/obibar x bat/fish/shark
- Minimum read count cut-off settings (0, 10, 40, 60)
- Dataset metadata distinguishing samples (e.g. the five marine samples of the fish dataset)

## Outputs

- alpha_beta_diversity_results.tsv containing per-sample ASV richness, Jaccard dissimilarity estimates, and t-test results
- Boxplots of alpha- and beta-diversity per pipeline and dataset (bxplt_diversity_intraspecies.R-style)

## How to apply

Load the nine per-pipeline final taxa tables (vtam/dalu/obibar x bat/fish/shark, e.g. dalu_bat_final_taxa.tsv) from out/summary_min_readcout_{0,10,40,60}/ and compute per-sample ASV richness for each pipeline/dataset combination. Compute Jaccard distances between samples with the vegan package in R, and compare alpha- and beta-diversity estimates between pipelines with t-tests at minimum read count cut-offs 0, 10, 40, and 60 (VTAM-optimized cut-offs were 10 for fish, 40 for shark, 60 for bat). Inspect the five marine fish samples of the fish dataset to confirm that unfiltered tag-jump false positives in DALU/OBIbaR inflate richness and deflate inter-sample dissimilarity. Write diversity estimates and t-test results to alpha_beta_diversity_results.tsv and produce boxplots following the logic of the bxplt_diversity_intraspecies.R script.

## Related tools

- **VTAM** (Produces the control-validated final taxa tables whose ASV richness is compared; its filtering yields the VTAM baseline pipeline results) — https://github.com/aitgon/vtam
- **R** (Computes per-sample ASV richness, t-tests between pipelines, and diversity boxplots)
- **vegan** (R package used to compute Jaccard distances between samples (beta-diversity))
- **vtam_benchmark** (Snakemake/Singularity repository reproducing the benchmark that generated the per-pipeline output tables) — https://github.com/aitgon/vtam_benchmark

## Examples

```
Rscript -e 'library(vegan); d <- vegdist(t(asv_table), method="jaccard")'
```

## Evaluation signals

- T-tests show significantly higher ASV richness in DALU and OBIbaR relative to VTAM: in the bat dataset for all read count cut-offs, and in the fish dataset using 0 and 10 cut-offs
- Jaccard-based inter-sample dissimilarity is correspondingly lower for DALU and OBIbaR than for VTAM
- Review of the five marine fish samples confirms unfiltered tag-jump false positives inflate DALU/OBIbaR richness and deflate dissimilarity
- Results are reported consistently across all four minimum read count cut-offs (0, 10, 40, 60)
- alpha_beta_diversity_results.tsv is produced with both diversity estimates and t-test results

## Limitations

- Unfiltered tag-jump false positives in DALU/OBIbaR inflate per-sample richness and deflate inter-sample dissimilarity, so richness differences do not necessarily indicate better recovery of true taxa
- Software versions of DADA2, OBITools, VTAM, and metabaR were not stated in the comparison
- DALU and OBIbaR pipelines are referenced by name only, without tool identity, version, or hosting location

## Evidence
