---
name: microbiome-metric-aggregation-and-comparison
description: Use when per-dataset alpha diversity metric tables from QIIME2's diversity
  module must be merged with metadata into one table, or when metric conclusions must
  be re-validated under a second condition such as rarefaction.
when_to_use_negative:
- Input is raw sequencing reads rather than computed alpha diversity tables — the
  reads must first be denoised (DEBLUR) and metrics calculated via QIIME2's diversity
  module
- The analysis goal is taxonomic beta diversity or differential abundance rather than
  alpha diversity metric behavior across the Richness, Dominance, Phylogenetics, and
  Information categories
- Projects lack associated metadata, since projects without metadata were excluded
  from the aggregation
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: QIIME2
  role: Computes the 19 alpha diversity metrics via its diversity module and processes
    samples following QIIME2 best practices
  repo: https://qiime2.org/
- name: QIIME2 2022.2.0
  role: Specific workflow version used for denoising and diversity metric calculation,
    compatible with newer versions
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment in which QIIME2 sample processing and diversity analyses
    are run
- name: Jupyter
  role: Notebooks used to load aggregated tables, explore data, and produce visualizations
    of metric comparisons
- name: sra-tools
  role: Downloads the source stool-sample datasets from SRA before metric computation
    and aggregation
  repo: https://github.com/ncbi/sra-tools
- name: DEBLUR
  role: Denoising tool producing the feature tables on which the aggregated alpha
    metrics are based
merged_aliases:
- cross-study-microbiome-metrics-aggregation-with-metadata
merged_alias_records:
- alias: cross-study-microbiome-metrics-aggregation-with-metadata
  slug: cross-study-microbiome-metrics-aggregation-with-metadata
  cosine_score: 0.8249
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:3377bdde1dfcddebfc45e31a5a0b6b0d74780c63def13604cd10fbb5f8115418
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DEBLUR
  - Jupyter
  - QIIME 2
  - QIIME2
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# microbiome-metric-aggregation-and-comparison

## Summary

Aggregate per-dataset QIIME2 alpha diversity metric tables (alpha-out_dataset-name.tsv) with sample metadata into a combined per-sample table, then compare metric values across conditions (e.g., rarefied vs non-rarefied) to validate metric behavior.

## When to use

Use when per-dataset alpha diversity metric tables from QIIME2's diversity module must be merged with metadata into one table, or when metric conclusions must be re-validated under a second condition such as rarefaction.

## When NOT to use

- Input is raw sequencing reads rather than computed alpha diversity tables — the reads must first be denoised (DEBLUR) and metrics calculated via QIIME2's diversity module
- The analysis goal is taxonomic beta diversity or differential abundance rather than alpha diversity metric behavior across the Richness, Dominance, Phylogenetics, and Information categories
- Projects lack associated metadata, since projects without metadata were excluded from the aggregation

## Inputs

- Per-dataset alpha diversity metric tables (alpha-out_dataset-name.tsv) from QIIME2's diversity module
- Per-sample metadata for each of the 13 stool-sample projects
- Rarefied feature tables from the repository
- DEBLUR-denoised feature tables for the 4,596 stool samples

## Outputs

- Combined per-sample table aggregated.tsv / alpha_aggregated.tsv containing all 19 alpha diversity metrics plus metadata
- Rarefied-vs-non-rarefied comparison table (rarefied_vs_nonrarefied_alpha_metrics.tsv)
- Notebook-based visualizations and correlation analyses of metric behavior

## How to apply

For each dataset, export the 19 alpha diversity metrics computed by QIIME2's diversity module into datasets/dataset-name/alpha-out_dataset-name.tsv, then combine these tables with each dataset's sample metadata into a single per-sample file (aggregated.tsv / alpha_aggregated.tsv). Load these aggregated files plus rarefied feature tables into Jupyter notebooks (environment built from requirements.txt, QIIME2 2022.2.0 under anaconda or a Python venv). Recompute all metrics on rarefied data, align the rarefied values with non-rarefied results per sample and dataset, and compare metric-by-metric across the 13 stool-sample projects (4,596 samples). Confirm that conclusions for the four metric categories (Richness, Dominance, Phylogenetics, Information) hold under both conditions, since non-rarefied data was used to preserve as much information as possible.

## Related tools

- **QIIME2** (Computes the 19 alpha diversity metrics via its diversity module and processes samples following QIIME2 best practices) — https://qiime2.org/
- **QIIME2 2022.2.0** (Specific workflow version used for denoising and diversity metric calculation, compatible with newer versions) — https://qiime2.org/
- **anaconda** (Virtual environment in which QIIME2 sample processing and diversity analyses are run)
- **Jupyter** (Notebooks used to load aggregated tables, explore data, and produce visualizations of metric comparisons)
- **sra-tools** (Downloads the source stool-sample datasets from SRA before metric computation and aggregation) — https://github.com/ncbi/sra-tools
- **DEBLUR** (Denoising tool producing the feature tables on which the aggregated alpha metrics are based)

## Examples

```
qiime diversity alpha --i-table rarefied_table.qza --p-metrics observed_features,shannon,faith_pd --o-alpha-diversity alpha-out_dataset-name.tsv
```

## Evaluation signals

- The aggregated table covers all 4,596 stool samples from the 13 publicly available human microbiome projects
- The aggregated table contains all 19 alpha diversity metrics as columns
- Metric-by-metric comparison of rarefied vs non-rarefied values shows conclusions for the four categories (Richness, Dominance, Phylogenetics, Information) are unchanged
- Each dataset's per-sample table alpha-out_dataset-name.tsv aligns one-to-one with its metadata when combined
- Notebook runs reproduce without errors using the Python venv built from requirements.txt

## Limitations

- Raw sample data is not shared in the repository due to size — only accession numbers (SraRunTable.txt) are provided in each dataset's folder, so full reproduction requires re-downloading from SRA
- Rarefied dataset availability is only described as 'data available in repository' without an explicit accession or location
- ACE and Faith metrics could not be recalculated on synthetic validation datasets due to technical limitations
- Sequencing files were processed with QIIME2 on a conda environment without version numbers reported for DADA2 or DEBLUR, so denoising may be a source of variability in estimated richness

## Evidence

- [readme] Per-dataset alpha tables aggregated with metadata: "All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata."
- [readme] Metrics computed with QIIME2 diversity module: "All alpha diversity metrics were caluclated using QIIME2's diversity module."
- [results] Non-rarefied calculation with rarefied validation: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [results] Coverage of 4,596 samples across 13 projects: "calculated for the sequence datasets obtained for a total of 4,596 stool samples included in 13 publicly available human microbiome projects"
- [results] Four metric categories: "Alpha metrics used in 68 microbiome studies or in the QIIME 2 suite were grouped into four categories"
