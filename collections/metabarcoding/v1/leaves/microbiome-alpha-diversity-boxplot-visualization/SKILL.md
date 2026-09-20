---
name: microbiome-alpha-diversity-boxplot-visualization
description: Use when you have aggregated alpha diversity tables (alpha-out_dataset-name.tsv
  or alpha_aggregated.tsv) with sample metadata and need to compare metric distributions
  between microbiome projects, e.g. reproducing Fig. 4 for the 471_swedish and 514_imp_project
  cohorts.
when_to_use_negative:
- Input is raw sequencing data (SRA runs) that still needs downloading with sra-tools
  and QIIME2 processing/denoising — metric values must already exist in the aggregated
  TSV tables
- The goal is beta diversity (between-sample) analysis rather than within-sample alpha
  diversity metric comparison
- Projects lacking associated metadata must be excluded rather than plotted, per the
  paper's filter steps
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QIIME2
  role: Source of the alpha diversity metric values via QIIME2's diversity module,
    which were aggregated into the TSV tables visualized here
  repo: https://qiime2.org/
- name: Jupyter
  role: Notebooks in which the aggregated tables are loaded with pandas, subset by
    project code, and boxplots are rendered and saved as PNG
- name: anaconda
  role: Virtual environment platform on which the QIIME2 processing that produced
    the alpha tables was run
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:3516b1889664e7999c1174d7a30dc8028967d981919faedab2c4799b07c6c110
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - QIIME2
  - anaconda
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# Microbiome alpha diversity boxplot visualization

## Summary

Generates per-project boxplots of recommended alpha diversity metrics (Shannon, Observed features, Berger-Parker) from the aggregated QIIME2 alpha diversity tables, reproducing Fig. 4-style comparisons across human gut microbiome projects. It complements the correlation analysis by making richness, entropy and dominance differences between cohorts visually explicit.

## When to use

Use when you have aggregated alpha diversity tables (alpha-out_dataset-name.tsv or alpha_aggregated.tsv) with sample metadata and need to compare metric distributions between microbiome projects, e.g. reproducing Fig. 4 for the 471_swedish and 514_imp_project cohorts.

## When NOT to use

- Input is raw sequencing data (SRA runs) that still needs downloading with sra-tools and QIIME2 processing/denoising — metric values must already exist in the aggregated TSV tables
- The goal is beta diversity (between-sample) analysis rather than within-sample alpha diversity metric comparison
- Projects lacking associated metadata must be excluded rather than plotted, per the paper's filter steps

## Inputs

- alpha-out_dataset-name.tsv files (per-dataset QIIME2 alpha diversity metrics aggregated per sample) for all 4,596 stool samples from 13 publicly available human microbiome projects
- alpha_aggregated.tsv (combined alpha diversity metrics for all samples with metadata)
- Sample metadata including Table S3 project codes (e.g. 471_swedish, 514_imp_project)
- requirements.txt from the mauroibz/alpha-diversities repository

## Outputs

- PNG boxplot figure of recommended alpha diversity metrics (Observed features, Shannon, Berger-Parker) per project, reproducing Fig. 4
- PNG correlation panel (Pearson and Spearman) of the metrics, reproducing Supplementary Fig. S5 (computed in the same notebook workflow)

## How to apply

Build a Python virtual environment and install pinned dependencies with pip install -r requirements.txt from the mauroibz/alpha-diversities repository, then run the Jupyter notebooks as the kernel of choice. Load the aggregated alpha diversity tables (alpha_aggregated.tsv or per-dataset alpha-out_dataset-name.tsv) with pandas, which combine QIIME2 diversity-module metrics for all 4,596 stool samples from 13 publicly available human microbiome projects with each dataset's metadata. Subset samples to the two projects of interest using the Table S3 project codes (471_swedish and 514_imp_project) found in the metadata column. Generate boxplots of the recommended alpha diversity metrics per project — Observed features (Richness), Shannon (Information/entropy, which summarizes both richness and dominance), and Berger-Parker (Dominance) — and save the resulting figure as a PNG. Validation requires that plotted distributions match the metric values in the aggregated alpha diversity tables and reproduce the Fig. 4 boxplot shapes for the two named projects.

## Related tools

- **QIIME2** (Source of the alpha diversity metric values via QIIME2's diversity module, which were aggregated into the TSV tables visualized here) — https://qiime2.org/
- **Jupyter** (Notebooks in which the aggregated tables are loaded with pandas, subset by project code, and boxplots are rendered and saved as PNG)
- **anaconda** (Virtual environment platform on which the QIIME2 processing that produced the alpha tables was run)

## Examples

```
python -m venv venv/ && source venv/bin/activate && pip install -r requirements.txt && jupyter notebook
```

## Evaluation signals

- The generated PNG reproduces the Fig. 4 boxplot distributions for projects 471_swedish and 514_imp_project (per Table S3 codes)
- Every plotted metric value matches the corresponding entry in the aggregated alpha diversity tables (alpha-out_dataset-name.tsv / alpha_aggregated.tsv)
- The companion correlation panel reproduces the metric relationships shown in Supplementary Fig. S5 (Pearson's linear and Spearman's rank coefficients)
- The subset contains only samples belonging to the two named project codes, with no samples from other projects leaking in
- The notebook environment matches the pinned requirements.txt and runs cells in sequential order

## Limitations

- Supplementary material (Table S1, S2, S3, Figure S5) is referenced but only a placeholder link is provided, so project codes and reference figures may need manual retrieval
- Raw sample data is not shared through the repository due to size; only accession numbers are available within each dataset's folder, so metric tables must be regenerated or obtained from Zenodo (10.5281/zenodo.8170289)
- All results were computed with non-rarefied data (to preserve information), with rarefied validation data only loosely described as 'available in repository'; plotting rarefied vs non-rarefied values may differ
- A hypothesized subsampling artifact in the KarenThai cohort means some per-project distributions may reflect sampling artifacts rather than true diversity differences

## Evidence

- [readme] All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata.: "All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata."
- [readme] Exploration of data and vizualizations were carried out on different jupyter notebooks. Package requirements are on requirements.txt.: "Exploration of data and vizualizations were carried out on different jupyter notebooks. Package requirements are on requirements.txt."
- [results] Pearson's linear correlation coefficient and Spearman 's rank correlation coefficients were calculated to identify correlations among metrics: [evidence span withheld — anchor status: not_found]
- [discussion] utilizing data from two analyzed studies (projects 471_swedish and 514_imp_project, according to Table S3 codes): "utilizing data from two analyzed studies (projects 471_swedish and 514_imp_project, according to Table S3 codes)"
- [abstract] Key metrics that should be required in microbiome analysis include richness, phylogenetic diversity, entropy, dominance of a few microbes over others, and an estimate of unobserved microbes.: "Key metrics that should be required in microbiome analysis include richness, phylogenetic diversity, entropy, dominance of a few microbes over others, and an estimate of unobserved microbes."
