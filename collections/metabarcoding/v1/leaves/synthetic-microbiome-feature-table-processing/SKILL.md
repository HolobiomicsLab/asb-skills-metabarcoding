---
name: synthetic-microbiome-feature-table-processing
description: Use when validating alpha diversity metric behavior on synthetic datasets
  with controlled community composition, e.g. testing whether dominance metrics respond
  to artificially imposed unevenness ratios.
when_to_use_negative:
- The input is real sequencing data from SRA rather than a synthetic controlled-composition
  table — use the standard QIIME2 best-practices processing workflow with DADA2/DEBLUR
  instead
- You need ACE or Faith's phylogenetic diversity values — these cannot be recalculated
  on synthetic datasets due to technical limitations
- You plan to rarefy before computing alpha metrics — the article's non-rarefied convention
  was chosen to preserve as much information as possible
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Hosts the feature-table artifacts and computes all alpha diversity metrics
    via its diversity module (qiime diversity alpha) on non-rarefied synthetic tables
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment manager in which QIIME2 is installed for all sample processing
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:556548b0a1f36c8ef2f36d509cb74be59585ebf0bcb4486f5aa37f25bb0220bf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# synthetic-microbiome-feature-table-processing

## Summary

Import controlled-composition synthetic microbiome feature tables (negative-binomial generated at 2x, 10x, and 100x ASV2/ASV1 unevenness ratios against an average of 0.8x) into QIIME2 as feature-table artifacts and recompute alpha diversity metrics on non-rarefied data to test metric sensitivity to community imbalance.

## When to use

Use when validating alpha diversity metric behavior on synthetic datasets with controlled community composition, e.g. testing whether dominance metrics respond to artificially imposed unevenness ratios.

## When NOT to use

- The input is real sequencing data from SRA rather than a synthetic controlled-composition table — use the standard QIIME2 best-practices processing workflow with DADA2/DEBLUR instead
- You need ACE or Faith's phylogenetic diversity values — these cannot be recalculated on synthetic datasets due to technical limitations
- You plan to rarefy before computing alpha metrics — the article's non-rarefied convention was chosen to preserve as much information as possible

## Inputs

- Three negative-binomial synthetic feature tables with ASV2/ASV1 unevenness ratios of 2x, 10x, and 100x (average ratio 0.8x)
- QIIME2-compatible feature-table artifacts
- requirements.txt with pinned dependencies for the analysis environment

## Outputs

- alpha-out_<dataset-name>.tsv per synthetic dataset, with one column per QIIME2 alpha metric (excluding ACE and Faith)
- Per-dataset alpha metric values compared across the 2x/10x/100x unevenness levels

## How to apply

Build the negative-binomial synthetic feature tables at ASV2/ASV1 unevenness ratios of 2x, 10x, and 100x (relative to an average of 0.8x) and import each into QIIME2 as a feature-table artifact. Run QIIME2's diversity module (qiime diversity alpha) on each table with non-rarefied data, since the authors computed metrics without rarefaction to preserve as much information as possible, computing every available alpha metric except ACE and Faith, which cannot be recalculated on synthetic datasets due to technical limitations. Export the resulting metric values and aggregate them per dataset into TSV tables named alpha-out_<dataset-name>.tsv, mirroring the article's datasets/dataset-name/alpha-out_dataset-name.tsv convention. Finally, compare recomputed metrics across the 2x, 10x, and 100x unevenness levels to confirm that dominance-category metrics increase with the imposed ASV2/ASV1 ratio.

## Related tools

- **QIIME2** (Hosts the feature-table artifacts and computes all alpha diversity metrics via its diversity module (qiime diversity alpha) on non-rarefied synthetic tables) — https://qiime2.org/
- **anaconda** (Virtual environment manager in which QIIME2 is installed for all sample processing)

## Examples

```
qiime diversity alpha --i-table synthetic_10x.qza --p-metric simpson --o-alpha-diversity alpha-diversity_10x.qza
```

## Evaluation signals

- An alpha-out TSV table exists for each of the three synthetic datasets (2x, 10x, 100x)
- Each TSV contains the QIIME2 alpha metric columns except ACE and Faith
- Dominance-category metrics increase monotonically with the imposed ASV2/ASV1 unevenness ratio
- Metric values were computed on non-rarefied feature tables, consistent with the article's convention
- Environment was built from the pinned requirements.txt so recomputation matches the article's tool versions

## Limitations

- ACE and Faith cannot be recalculated on synthetic datasets due to technical limitations, so they are absent from the output TSVs
- Faith's metric is algorithmic and 'continuously suffering changes', which motivated excluding its formula from reproducible recalculation
- Findings were computed with non-rarefied data; rarefied validation results are only described as 'data available in repository' without an explicit accession or location
- Only 7 synthetic datasets were used in the article's validation, limiting generalization beyond controlled unevenness scenarios

## Evidence

- [results] synthetic validation scope: "we validated our findings by applying the alpha diversity metrics to 7 synthetics datasets"
- [results] non-rarefied computation convention: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them."
- [discussion] ACE and Faith exclusion on synthetic data: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
- [readme] QIIME2 computation and TSV aggregation convention: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
