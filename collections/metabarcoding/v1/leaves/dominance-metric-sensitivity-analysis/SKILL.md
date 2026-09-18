---
name: dominance-metric-sensitivity-analysis
description: Use when you need to test whether dominance-type alpha diversity metrics
  (e.g., dominance, Gini, Simpson) respond monotonically to artificially imposed community
  unevenness, using synthetic ASV feature tables with known abundance ratios.
when_to_use_negative:
- The dataset is a real (empirical) microbiome feature table where phylogenetic metrics
  like Faith or unobserved-richness metrics like ACE can be computed — these are excluded
  here only because of technical limitations on synthetic data
- The goal is rarefaction-based comparison — this skill computes metrics on non-rarefied
  data, and rarefied validation would require a different (unpublished-location) dataset
- The input is raw sequence data (e.g., SRA FASTQ) rather than an already-denoised
  feature table — sample processing with DADA2/DEBLUR is a separate workflow
edam_operation: http://edamontology.org/operation_2123
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Import synthetic feature tables and compute alpha diversity metrics via its
    diversity module ('qiime diversity alpha') on non-rarefied data
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment in which QIIME2 is installed and executed for the synthetic-dataset
    recomputation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:4af5f03a56fb034376e383a1195e0cce8656a32808a1155bddeb9c21ae3c947d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - anaconda
  edam_operation: http://edamontology.org/operation_2123
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# dominance-metric-sensitivity-analysis

## Summary

Validates the behavior of dominance-category alpha diversity metrics by recomputing them with QIIME2's diversity module on negative-binomial synthetic feature tables with controlled unevenness (ASV2/ASV1 ratios of 2x, 10x, and 100x against an average of 0.8x), demonstrating their sensitivity to community imbalance.

## When to use

Use when you need to test whether dominance-type alpha diversity metrics (e.g., dominance, Gini, Simpson) respond monotonically to artificially imposed community unevenness, using synthetic ASV feature tables with known abundance ratios.

## When NOT to use

- The dataset is a real (empirical) microbiome feature table where phylogenetic metrics like Faith or unobserved-richness metrics like ACE can be computed — these are excluded here only because of technical limitations on synthetic data
- The goal is rarefaction-based comparison — this skill computes metrics on non-rarefied data, and rarefied validation would require a different (unpublished-location) dataset
- The input is raw sequence data (e.g., SRA FASTQ) rather than an already-denoised feature table — sample processing with DADA2/DEBLUR is a separate workflow

## Inputs

- Three negative-binomial synthetic feature tables with ASV2/ASV1 abundance ratios of 2x, 10x, and 100x against an average of 0.8x
- QIIME2 feature-table artifacts (.qza) imported from the synthetic tables
- requirements.txt from the mauroibz/alpha-diversities repository

## Outputs

- Per-dataset alpha diversity TSV tables named alpha-out_<dataset-name>.tsv containing all QIIME2 alpha metric columns except ACE and Faith
- Recomputed metric values across the 2x, 10x, and 100x unevenness levels for sensitivity comparison

## How to apply

Build a Python virtual environment and install the dependencies pinned in requirements.txt from the mauroibz/alpha-diversities repository. Import each negative-binomial synthetic feature table (ASV2/ASV1 ratios of 2x, 10x, and 100x against an average of 0.8x) into QIIME2 as feature-table artifacts, then run 'qiime diversity alpha' on each table using non-rarefied data to preserve as much information as possible, computing every available alpha metric except ACE and Faith, which cannot be recalculated on synthetic datasets due to technical limitations. Export the resulting metric values and aggregate them per dataset into TSV tables named alpha-out_<dataset-name>.tsv, following the repository's convention for alpha diversity tables. Compare the recomputed dominance-category metrics across the 2x, 10x, and 100x unevenness levels to reproduce the reported sensitivity of dominance metrics to artificial community imbalance.

## Related tools

- **QIIME2** (Import synthetic feature tables and compute alpha diversity metrics via its diversity module ('qiime diversity alpha') on non-rarefied data) — https://qiime2.org/
- **anaconda** (Virtual environment in which QIIME2 is installed and executed for the synthetic-dataset recomputation)

## Examples

```
qiime diversity alpha --i-table synthetic-100x.qza --p-metric dominance --o-alpha-diversity alpha-100x.qza
```

## Evaluation signals

- An alpha-out TSV table exists for each of the three synthetic datasets (2x, 10x, 100x)
- Each TSV contains the QIIME2 alpha metric columns except ACE and Faith
- Dominance-category metrics increase monotonically with the imposed ASV2/ASV1 ratio (2x < 10x < 100x)
- Metrics were computed on non-rarefied feature tables, matching the paper's stated approach

## Limitations

- ACE and Faith cannot be recalculated on synthetic datasets due to technical limitations, so the comparison excludes these two metrics
- Results are only validated here for the three unevenness ratios (2x, 10x, 100x) against an average of 0.8x; the paper's broader claims were additionally checked on rarefied datasets whose repository location is not explicitly specified
- The paper's workflows ran on QIIME 2022.2.0; newer versions should be compatible but are unverified

## Evidence

- [results] dominance metrics synthetic sensitivity: "we validated our findings by applying the alpha diversity metrics to 7 synthetics datasets"
- [discussion] ace-faith-exclusion: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
- [readme] qiime-diversity-module: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
- [results] non-rarefied-computation: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them."
