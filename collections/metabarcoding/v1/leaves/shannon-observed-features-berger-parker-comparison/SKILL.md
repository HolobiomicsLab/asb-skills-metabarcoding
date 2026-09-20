---
name: shannon-observed-features-berger-parker-comparison
description: 'Use when you need to justify a minimal alpha diversity reporting set
  for microbiome data: quantify how Shannon, Observed features, and Berger-Parker
  relate (correlations, Supplementary Fig. S5) and compare per-project distributions
  for specific cohorts.'
when_to_use_negative:
- Input is raw sequencing reads rather than pre-computed alpha diversity values —
  first process samples with QIIME2's diversity module before applying this comparison
  workflow
- The question concerns phylogenetic diversity (e.g., Faith) or unobserved microbe
  estimation rather than the richness/dominance/entropy categories this skill compares
- Data are rarefied-only results you intend to present as primary — the paper calculated
  metrics on non-rarefied data to preserve information and used rarefied data only
  for validation
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Diversity module used to compute the alpha diversity metrics (Shannon, Observed
    features, Berger-Parker) that feed this comparison
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment used for QIIME2 sample processing and analysis
- name: Jupyter
  role: Notebooks in the alpha-diversities repository used for data exploration, correlation
    computation, and figure generation
  repo: https://github.com/mauroibz/alpha-diversities
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:ec0bcc7038e4c94991d3302efce162c93edfc814c71cbd6631136dda8be554be
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - QIIME2
  - anaconda
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# shannon-observed-features-berger-parker-comparison

## Summary

A correlation-and-visualization workflow that validates the recommended minimal alpha diversity metric set from Cassol et al. (2025) by computing Pearson and Spearman correlations among Shannon (Information), Observed features (Richness), and Berger-Parker (Dominance) across 4,596 stool samples, then reproducing Fig. 4 boxplots for projects 471_swedish and 514_imp_project.

## When to use

Use when you need to justify a minimal alpha diversity reporting set for microbiome data: quantify how Shannon, Observed features, and Berger-Parker relate (correlations, Supplementary Fig. S5) and compare per-project distributions for specific cohorts.

## When NOT to use

- Input is raw sequencing reads rather than pre-computed alpha diversity values — first process samples with QIIME2's diversity module before applying this comparison workflow
- The question concerns phylogenetic diversity (e.g., Faith) or unobserved microbe estimation rather than the richness/dominance/entropy categories this skill compares
- Data are rarefied-only results you intend to present as primary — the paper calculated metrics on non-rarefied data to preserve information and used rarefied data only for validation

## Inputs

- Aggregated alpha diversity table alpha_aggregated.tsv (Shannon, Observed features, Berger-Parker plus other metrics for all 4,596 stool samples)
- Per-dataset alpha diversity tables alpha-out_dataset-name.tsv combined with each dataset's metadata
- Sample metadata with Table S3 project codes (471_swedish, 514_imp_project)
- requirements.txt pinning the notebook environment

## Outputs

- Correlation panel PNG reproducing Supplementary Fig. S5 (Pearson and Spearman coefficients among Shannon, Observed features, Berger-Parker)
- Boxplot PNG reproducing Fig. 4 (per-project distributions of recommended alpha diversity metrics for 471_swedish and 514_imp_project)

## How to apply

Build a Python virtual environment and install the pinned dependencies with 'pip install -r requirements.txt' from the mauroibz/alpha-diversities repository. Load the aggregated alpha diversity tables (alpha-out_dataset-name.tsv, combined as alpha_aggregated.tsv, with sample metadata for all 4,596 samples) into pandas in Jupyter notebooks. Compute Pearson's linear and Spearman's rank correlation coefficients among Shannon, Observed features, and Berger-Parker, and render the correlation panel reproducing Supplementary Fig. S5. Subset samples to projects coded 471_swedish and 514_imp_project according to Table S3 project codes, generate per-project boxplots of the recommended metrics to regenerate Fig. 4, and save both figures as PNG files. The rationale is the paper's finding that Shannon serves as a single parameter that summarizes and weighs both the richness and dominance categories, making it the Information-category representative alongside Observed features and Berger-Parker.

## Related tools

- **QIIME2** (Diversity module used to compute the alpha diversity metrics (Shannon, Observed features, Berger-Parker) that feed this comparison) — https://qiime2.org/
- **anaconda** (Virtual environment used for QIIME2 sample processing and analysis)
- **Jupyter** (Notebooks in the alpha-diversities repository used for data exploration, correlation computation, and figure generation) — https://github.com/mauroibz/alpha-diversities

## Examples

```
python -m venv venv/; source venv/bin/activate; pip install -r requirements.txt; jupyter notebook
```

## Evaluation signals

- The correlation panel must reproduce the metric relationships shown in Supplementary Fig. S5 for Pearson's and Spearman's coefficients among Shannon, Observed features, and Berger-Parker
- Fig. 4 boxplot distributions must match those shown for the two named projects 471_swedish and 514_imp_project using Table S3 codes
- All plotted values must match those in the aggregated alpha diversity tables (alpha-out_dataset-name.tsv / alpha_aggregated.tsv) for the 4,596 samples
- The recommended metric set interpretation must hold: Shannon should summarize and weigh both the richness and dominance categories together

## Limitations

- Raw sample data are not shared in the repository due to size; only accession numbers are provided within each dataset's folder (SraRunTable.txt or similar)
- Supplementary material (Table S3 project codes, Figure S5) is referenced but only a placeholder link is provided, complicating exact reproduction
- Alpha diversity metrics were calculated with non-rarefied data; rarefied dataset availability is only described as 'data available in repository' without explicit accession or location

## Evidence

- [discussion] the paper recommends reporting the Shannon value because it not only informs entropy but also serves as a single parameter that summarizes and weighs both the richness and dominance categories together: [evidence span withheld — anchor status: not_found]
- [results] Pearson's linear correlation coefficient and Spearman 's rank correlation coefficients were calculated to identify correlations among metrics: [evidence span withheld — anchor status: not_found]
- [readme] All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata.: "All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv"
- [results] calculated for the sequence datasets obtained for a total of 4,596 stool samples included in 13 publicly available human microbiome projects: "calculated for the sequence datasets obtained for a total of 4,596 stool samples included in 13 publicly available human microbiome projects"
- [readme] Build a python virtual enviroment ... Install requirments > pip install -r requirements.txt: "Install requirments > pip install -r requirements.txt"
