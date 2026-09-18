---
name: qiime2-alpha-diversity-table-handling
description: Use when alpha diversity metrics have been computed with QIIME2's diversity
  module and need to be aggregated per dataset, combined with metadata, and visualized/compared
  (e.g., correlations, boxplots per project).
when_to_use_negative:
- Input is still raw sequencing data (SRA runs) — sample processing and metric computation
  with QIIME2's diversity module must happen first
- The analysis requires ACE or Faith recomputation on synthetic datasets — the paper
  reports these could not be recalculated due to technical limitations
- Only rarefied data is acceptable without validation — the source study's primary
  results use non-rarefied data with rarefied validation, so conclusions should not
  rest on rarefied tables alone
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: QIIME2
  role: Compute all alpha diversity metrics via its diversity module; version 2022.2.0
    used in the study
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment hosting the QIIME2 processing and analysis stack
- name: sra-tools
  role: Download the 13 publicly available human microbiome project datasets from
    SRA prior to metric computation
  repo: https://github.com/ncbi/sra-tools
- name: Jupyter
  role: Notebook environment (with pandas) for loading aggregated alpha tables, computing
    correlations, and rendering figures
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:0285db92b7a10d68f03733974136b4b6f241c1dacc420f8444183234dfeab0de
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - QIIME2
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# qiime2-alpha-diversity-table-handling

## Summary

Aggregate QIIME2-computed alpha diversity metrics per dataset into alpha-out_dataset-name.tsv files, combine all 4,596 samples into alpha_aggregated.tsv with sample metadata, and explore/visualize the metrics (Shannon, Observed features, Berger-Parker, Faith) in Jupyter notebooks to reproduce correlation panels and boxplots.

## When to use

Use when alpha diversity metrics have been computed with QIIME2's diversity module and need to be aggregated per dataset, combined with metadata, and visualized/compared (e.g., correlations, boxplots per project).

## When NOT to use

- Input is still raw sequencing data (SRA runs) — sample processing and metric computation with QIIME2's diversity module must happen first
- The analysis requires ACE or Faith recomputation on synthetic datasets — the paper reports these could not be recalculated due to technical limitations
- Only rarefied data is acceptable without validation — the source study's primary results use non-rarefied data with rarefied validation, so conclusions should not rest on rarefied tables alone

## Inputs

- alpha-out_dataset-name.tsv per-dataset alpha diversity tables (QIIME2 diversity module output)
- alpha_aggregated.tsv combined table of all metrics for all samples with sample metadata
- Sample metadata with Table S3 project codes (e.g., 471_swedish, 514_imp_project)
- requirements.txt for the pinned Python notebook dependencies

## Outputs

- Aggregated alpha diversity tables per dataset (alpha-out_dataset-name.tsv) merged with metadata
- Pearson/Spearman correlation panel reproducing Supplementary Fig. S5 (PNG)
- Boxplots of recommended metrics (Shannon, Observed features, Berger-Parker) per project reproducing Fig. 4 (PNG)

## How to apply

After processing each dataset through QIIME2 (2022.2.0 in the source study, DADA2/DEBLUR denoised), compute all alpha diversity metrics with QIIME2's diversity module on non-rarefied data to preserve as much information as possible, then validate conclusions on rarefied datasets as a check. Aggregate each dataset's metrics into datasets/dataset-name/alpha-out_dataset-name.tsv and combine with sample metadata into a single alpha_aggregated.tsv. Build a Python virtual environment (python -m venv venv/; pip install -r requirements.txt), load the aggregated tables into pandas in Jupyter notebooks, and compute Pearson's linear and Spearman's rank correlations among metrics to reproduce the Supplementary Fig. S5 correlation panel. For project-level comparisons, subset samples by the Table S3 project codes (e.g., 471_swedish and 514_imp_project) and generate boxplots of the recommended metrics to regenerate Fig. 4, saving figures as PNG files.

## Related tools

- **QIIME2** (Compute all alpha diversity metrics via its diversity module; version 2022.2.0 used in the study) — https://qiime2.org/
- **anaconda** (Virtual environment hosting the QIIME2 processing and analysis stack)
- **sra-tools** (Download the 13 publicly available human microbiome project datasets from SRA prior to metric computation) — https://github.com/ncbi/sra-tools
- **Jupyter** (Notebook environment (with pandas) for loading aggregated alpha tables, computing correlations, and rendering figures)

## Examples

```
source venv/bin/activate && pip install -r requirements.txt && jupyter notebook
```

## Evaluation signals

- The correlation panel must reproduce the metric relationships shown in Supplementary Fig. S5 (Pearson and Spearman among Shannon, Observed features, Berger-Parker)
- Boxplots for projects 471_swedish and 514_imp_project must match the Fig. 4 distributions for the recommended metrics
- Plotted values must match those in the aggregated alpha diversity tables (alpha-out_dataset-name.tsv / alpha_aggregated.tsv)
- Total sample count across datasets should equal 4,596 stool samples from the 13 publicly available projects
- Notebook environment must be reproducible: python -m venv with requirements.txt installed and used as the Jupyter kernel, cells run sequentially

## Limitations

- Raw sample data is not shared in the repository due to size; only accession numbers (SraRunTable.txt) are provided in each dataset's folder
- Supplementary material (Tables S1–S3, Figure S5) is referenced but only a placeholder link is provided
- ACE and Faith metrics could not be recalculated on synthetic datasets due to technical limitations
- Results were primarily computed on non-rarefied data; rarefied validation data is only described as 'data available in repository' without explicit location
- QIIME 2022.2.0 was used but should be compatible with newer versions; no version numbers were reported for DADA2 or DEBLUR

## Evidence

- [readme] All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata.: "All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata."
- [readme] To run the notebooks you can perform the following steps: Build a python virtual enviroment: "To run the notebooks you can perform the following steps:
    Build a python virtual enviroment
    > python -m venv venv/"
- [methods] All alpha diversity metrics were caluclated using QIIME2's diversity module.: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called alpha-out_dataset-name.tsv"
- [results] Pearson's linear correlation coefficient and Spearman's rank correlation coefficients were calculated to identify correlations among metrics: [evidence span withheld — anchor status: not_found]
- [results] alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [discussion] utilizing data from two analyzed studies (projects 471_swedish and 514_imp_project, according to Table S3 codes): "utilizing data from two analyzed studies (projects 471_swedish and 514_imp_project, according to Table S3 codes)"
- [intro] Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code.: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
