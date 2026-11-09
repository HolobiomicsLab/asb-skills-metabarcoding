---
name: chao1-richness-estimation
description: Use when estimating total microbial richness (including unobserved taxa)
  from amplicon feature tables where Chao1's singleton/doubleton assumptions hold,
  and when choosing a denoiser that does not remove singletons.
when_to_use_negative:
- The feature table was produced by DADA2, which removes singletons — Chao1's singleton-dependent
  estimator cannot recover unobserved richness and will collapse toward observed features.
- Data are rarefied and the goal is maximum information retention — the study calculated
  alpha metrics on non-rarefied data (rarefied tables were only used for validation).
- The metric of interest is not singleton-dependent richness (e.g. Faith's phylogenetic
  diversity or entropy metrics), where the singleton-treatment constraint does not
  apply.
tools:
- name: QIIME2 2022.2.0
  role: Suite in which the denoising and the diversity module used to compute Chao1
    are run
  repo: https://qiime2.org/
- name: DEBLUR
  role: Denoiser that retains singletons; provides the feature tables used for singleton-dependent
    alpha diversity analyses
- name: DADA2
  role: Contrast denoiser that discards singletons; used in the comparison to quantify
    Chao1 distortion
- name: sra-tools
  role: Downloads the SRA sequencing datasets used to build the feature tables
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:a43034052d336b946c1d0a2dba8d9992f3b3ed79f2c457c5d6095fa45b196ac2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# chao1-richness-estimation

## Summary

Estimates unobserved microbial richness with the Chao1 metric, which depends directly on singleton counts. Because DADA2 discards singletons while DEBLUR retains them, Chao1 must be computed on singleton-preserving denoised feature tables.

## When to use

Use when estimating total microbial richness (including unobserved taxa) from amplicon feature tables where Chao1's singleton/doubleton assumptions hold, and when choosing a denoiser that does not remove singletons.

## When NOT to use

- The feature table was produced by DADA2, which removes singletons — Chao1's singleton-dependent estimator cannot recover unobserved richness and will collapse toward observed features.
- Data are rarefied and the goal is maximum information retention — the study calculated alpha metrics on non-rarefied data (rarefied tables were only used for validation).
- The metric of interest is not singleton-dependent richness (e.g. Faith's phylogenetic diversity or entropy metrics), where the singleton-treatment constraint does not apply.

## Inputs

- Denoised QIIME2 feature table (artifact .qza) from a singleton-preserving denoiser such as DEBLUR
- Raw paired-end sequencing reads downloaded from SRA via sra-tools (accession numbers in datasets/xx_dataset_alias/SraRunTable.txt)
- Sample metadata table to join with per-sample alpha metrics

## Outputs

- Per-sample Chao1 values aggregated in datasets/dataset-name/alpha-out_dataset-name.tsv
- Combined metadata + alpha metrics table (alpha_aggregated.tsv)
- Singleton-removal impact table: dada2_chao1_robbins_comparison.tsv (per-sample and per-dataset DADA2-vs-DEBLUR deltas)

## How to apply

Denoise sequencing data with a denoiser that keeps singletons (DEBLUR, which per the study keeps singletons while DADA2 removes them), then compute Chao1 with QIIME2's diversity module (version 2022.2.0 in this workflow) on non-rarefied feature tables to preserve as much information as possible. Aggregate per-dataset metrics into datasets/dataset-name/alpha-out_dataset-name.tsv and combine with sample metadata (alpha_aggregated.tsv) following the mauroibz/alpha-diversities repository layout. To quantify the impact of singleton removal, compute per-sample and per-dataset deltas of Chao1 between DADA2 and DEBLUR runs and record them in dada2_chao1_robbins_comparison.tsv. Report the denoise tool used, since it is a source of variability for estimated richness.

## Related tools

- **QIIME2 2022.2.0** (Suite in which the denoising and the diversity module used to compute Chao1 are run) — https://qiime2.org/
- **DEBLUR** (Denoiser that retains singletons; provides the feature tables used for singleton-dependent alpha diversity analyses)
- **DADA2** (Contrast denoiser that discards singletons; used in the comparison to quantify Chao1 distortion)
- **sra-tools** (Downloads the SRA sequencing datasets used to build the feature tables) — https://github.com/ncbi/sra-tools

## Examples

```
qiime diversity alpha --i-table deblur_table.qza --p-metric chao1 --o-alpha-diversity chao1_vector.qza
```

## Evaluation signals

- Chao1 values are always >= observed features per sample; if Chao1 equals observed features, check whether the denoiser removed all singletons.
- Per-sample Chao1 values from DADA2 runs should show non-trivial negative deltas relative to the DEBLUR baseline when singletons were present pre-denoising.
- Output tables conform to the repository schema: per-dataset alpha-out_dataset-name.tsv, joined alpha_aggregated.tsv, and dada2_chao1_robbins_comparison.tsv for deltas.
- Sample IDs in the alpha metrics table match the dataset metadata after the join, with no missing-metadata projects included.
- Metrics were computed on non-rarefied tables (rarefied results only used for validation), consistent with the study's stated procedure.

## Limitations

- Chao1 computed on DADA2 output is invalid for estimating unobserved richness because DADA2 removes all singletons.
- The denoise tool is a source of variability for estimated microorganism richness and must be reported alongside the metric.
- The study hypothesizes a subsampling artifact in the KarenThai cohort according to Robbins, indicating cohort design can affect singleton-based metrics.
- Raw sample data are not shared in the repository due to size; only accession numbers are available in each dataset folder.

## Evidence

- [discussion] DADA2 removes singletons and DEBLUR keeps them.: "DADA2 removes singletons and DEBLUR keeps them."
- [results] Singletons required for some alpha metrics; DEBLUR used downstream: "all samples were processed using DADA2 and DEBLUR"
- [results] Non-rarefied tables used to preserve information: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them."
- [discussion] Denoise tool is a variability source for richness: "This difference should be considered as a source of variability when reporting the estimated microorganism richness of the sample, if the denoise tool is not reported."
- [readme] QIIME2 diversity module and aggregation layout: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
