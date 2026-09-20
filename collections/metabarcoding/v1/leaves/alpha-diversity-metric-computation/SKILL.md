---
name: alpha-diversity-metric-computation
description: Use when per-sample alpha diversity metrics (Shannon, Observed features,
  Berger-Parker, Chao1, Robbins, etc.) must be computed from QIIME2 feature tables
  derived from stool sequencing data, especially when rarefaction is in question or
  singleton-dependent metrics are needed.
when_to_use_negative:
- The only available feature tables were denoised with DADA2 — singletons are discarded,
  so singleton-dependent metrics (Chao1, Robbins) cannot be estimated; reprocess with
  DEBLUR first.
- The goal is beta diversity, ordination, or differential-abundance analysis rather
  than per-sample alpha metric computation.
- You need to recompute ACE or Faith on synthetic feature tables — the article reports
  these cannot be recalculated on synthetic datasets due to technical limitations.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: QIIME2 2022.2.0
  role: Diversity module used to compute all 19 alpha diversity metrics on non-rarefied
    feature tables
  repo: https://qiime2.org/
- name: DEBLUR
  role: Denoiser that retains singletons, required for singleton-dependent metrics
    (Chao1, Robbins); all downstream alpha analyses used DEBLUR data
- name: DADA2
  role: Alternative denoiser run for comparison only — removes singletons, making
    it unsuitable for Chao1/Robbins estimation
- name: sra-tools
  role: Downloads the 13 stool-study sequencing datasets from the SRA using accession
    numbers in each dataset's folder
  repo: https://github.com/ncbi/sra-tools
- name: anaconda
  role: Virtual environment hosting QIIME2 2022.2.0 for all sample processing
provenance:
  source_task_ids:
  - task_007
  - task_005
  - task_001
  - task_003
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:b761bcdcccac94b1fca3386266878ba518061bb98dda41a0ddb11fe5b43a0f46
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - Jupyter
  - QIIME 2
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# alpha-diversity-metric-computation

## Summary

Compute per-sample alpha diversity metrics (19 metrics spanning Richness, Dominance, Phylogenetics, and Information categories) from DEBLUR-denoised QIIME2 feature tables on non-rarefied stool microbiome data, then aggregate per-dataset metric tables with sample metadata. This mirrors the paper's empirical pipeline for characterizing and validating alpha metric behavior.

## When to use

Use when per-sample alpha diversity metrics (Shannon, Observed features, Berger-Parker, Chao1, Robbins, etc.) must be computed from QIIME2 feature tables derived from stool sequencing data, especially when rarefaction is in question or singleton-dependent metrics are needed.

## When NOT to use

- The only available feature tables were denoised with DADA2 — singletons are discarded, so singleton-dependent metrics (Chao1, Robbins) cannot be estimated; reprocess with DEBLUR first.
- The goal is beta diversity, ordination, or differential-abundance analysis rather than per-sample alpha metric computation.
- You need to recompute ACE or Faith on synthetic feature tables — the article reports these cannot be recalculated on synthetic datasets due to technical limitations.

## Inputs

- DEBLUR-denoised QIIME2 feature-table artifacts (.qza) per dataset
- Rarefied feature-table artifacts for validation runs
- SRA accession lists (SraRunTable.txt) from the Zenodo deposit 10.5281/zenodo.8170289
- Per-dataset sample metadata files
- requirements.txt from the mauroibz/alpha-diversities repository

## Outputs

- Per-dataset alpha metric tables: alpha-out_<dataset-name>.tsv with all 19 alpha diversity metric columns
- Combined per-sample table aggregated.tsv (alpha_aggregated.tsv) covering 4,596 stool samples from 13 human microbiome projects
- Rarefied vs non-rarefied comparison table (e.g., rarefied_vs_nonrarefied_alpha_metrics.tsv)
- DADA2 vs DEBLUR singleton-impact deltas for Chao1 and Robbins (dada2_chao1_robbins_comparison.tsv)

## How to apply

Set up a QIIME2 2022.2.0 environment in anaconda (or a Python venv with the repository's pinned requirements.txt), download the 13 stool-study datasets from SRA with sra-tools, and denoise with DEBLUR following QIIME2 best practices — DEBLUR must be used instead of DADA2 when singleton-dependent metrics like Chao1 and Robbins are needed, since DADA2 removes all singletons. Run QIIME2's diversity module (qiime diversity alpha) on non-rarefied feature tables to preserve as much information as possible, computing all 19 metrics (ACE and Faith excluded only when synthetic datasets are analyzed due to technical limitations). Export each dataset's metrics to datasets/dataset-name/alpha-out_dataset-name.tsv, then combine all per-dataset tables with sample metadata into a single aggregated table (alpha_aggregated.tsv / aggregated.tsv) covering all 4,596 stool samples. Validate the metric behavior by recomputing on rarefied tables and confirming that conclusions for the four metric categories (Richness, Dominance, Phylogenetics, Information) are unchanged.

## Related tools

- **QIIME2 2022.2.0** (Diversity module used to compute all 19 alpha diversity metrics on non-rarefied feature tables) — https://qiime2.org/
- **DEBLUR** (Denoiser that retains singletons, required for singleton-dependent metrics (Chao1, Robbins); all downstream alpha analyses used DEBLUR data)
- **DADA2** (Alternative denoiser run for comparison only — removes singletons, making it unsuitable for Chao1/Robbins estimation)
- **sra-tools** (Downloads the 13 stool-study sequencing datasets from the SRA using accession numbers in each dataset's folder) — https://github.com/ncbi/sra-tools
- **anaconda** (Virtual environment hosting QIIME2 2022.2.0 for all sample processing)

## Examples

```
qiime diversity alpha --i-table feature-table.qza --p-metric shannon --o-alpha-diversity alpha-shannon.qza
```

## Evaluation signals

- An alpha-out_<dataset-name>.tsv exists for each dataset and contains the QIIME2 alpha metric columns (all except ACE and Faith when synthetic datasets are analyzed).
- The aggregated table covers all 4,596 stool samples from the 13 publicly available human microbiome projects and contains all 19 alpha diversity metrics as columns.
- On the synthetic unevenness datasets (2x, 10x, 100x ASV2/ASV1 ratios), dominance-category metrics increase monotonically with the imposed ASV2/ASV1 ratio.
- Metric-by-metric comparison of rarefied vs non-rarefied recomputation across the 13 stool projects shows unchanged conclusions for all four metric categories.
- Chao1 and Robbins deltas between DADA2 and DEBLUR runs are nonzero and attributable to singleton removal, confirming the denoiser-dependence caveat.

## Limitations

- ACE and Faith could not be recalculated on synthetic datasets due to technical limitations; Faith's formula is also omitted from the paper because its implementation is algorithmic and 'continuously suffering changes'.
- DADA2-processed data cannot support singleton-dependent richness metrics (Chao1, Robbins); the denoise tool must be reported as a source of variability for estimated richness.
- Raw sample data is not shared in the repository due to size — only SRA accession numbers are provided, so recomputation requires re-downloading from SRA.
- The choice of metric set should be justified; the paper groups 19 metrics into four categories (Richness, Dominance, Phylogenetics, Information) and recommends a minimal set (Observed features, Shannon, Berger-Parker) rather than computing everything unthinkingly.

## Evidence

- [readme] All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv.: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
- [readme] Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code.: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [results] alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [discussion] DADA2 removes singletons and DEBLUR keeps them.: "DADA2 removes singletons and DEBLUR keeps them."
- [readme] Each dataset was downloaded from the SRA using sra-tools https://github.com/ncbi/sra-tools.: "Each dataset was downloaded from the SRA using sra-tools https://github.com/ncbi/sra-tools."
