---
name: cross-project-cohort-subsetting-by-metadata
description: Use when an analysis requires isolating one or more source studies from
  a combined multi-project microbiome alpha diversity table, guided by project-code
  annotations (e.g., Table S3 codes) present in the sample metadata.
when_to_use_negative:
- The input is a single-dataset alpha table (alpha-out_dataset-name.tsv) for one project
  only — no cross-project subsetting is needed
- The goal is a global correlation analysis (Supplementary Fig. S5) across all 4,596
  samples, where subsetting would discard data
- The target projects lack associated metadata, since the paper excluded projects
  lacking metadata
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3337
tools:
- name: QIIME2
  role: Produced the per-dataset alpha diversity tables (via its diversity module)
    that are aggregated and then subsetted by project code
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment manager used for the QIIME2 processing and notebook environment
    underlying the aggregated tables
- name: Jupyter
  role: Notebook environment where the aggregated table is loaded into pandas, subset
    by project codes, and plotted as boxplots
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:5a7c38e157996a44d3066ebe3b94e3f62661bd12c54aebb12cf015c6503ebb8f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - QIIME2
  - anaconda
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3337
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Cross-project cohort subsetting by metadata

## Summary

Extracts specific study cohorts (e.g., projects coded 471_swedish and 514_imp_project) from the paper's aggregated alpha diversity tables (alpha_aggregated.tsv / alpha-out_dataset-name.tsv) using Table S3 project codes in the sample metadata, enabling per-project comparisons such as the Fig. 4 boxplots of Shannon, Observed features, and Berger-Parker.

## When to use

Use when an analysis requires isolating one or more source studies from a combined multi-project microbiome alpha diversity table, guided by project-code annotations (e.g., Table S3 codes) present in the sample metadata.

## When NOT to use

- The input is a single-dataset alpha table (alpha-out_dataset-name.tsv) for one project only — no cross-project subsetting is needed
- The goal is a global correlation analysis (Supplementary Fig. S5) across all 4,596 samples, where subsetting would discard data
- The target projects lack associated metadata, since the paper excluded projects lacking metadata

## Inputs

- alpha_aggregated.tsv (combined alpha diversity metrics for all 4,596 samples merged with sample metadata)
- Per-dataset alpha diversity tables (datasets/dataset-name/alpha-out_dataset-name.tsv) with metadata
- Table S3 project-code mappings (e.g., 471_swedish, 514_imp_project, 248_citizen)

## Outputs

- Subsetted pandas DataFrame restricted to the target project codes
- Boxplots of Shannon, Observed features, and Berger-Parker per project (reproducing Fig. 4), saved as PNG files

## How to apply

Load the combined table alpha_aggregated.tsv (all 4,596 stool samples from 13 projects, built by aggregating per-dataset files alpha-out_dataset-name.tsv with each dataset's metadata) into pandas in a Jupyter notebook, with the virtual environment and pinned dependencies from requirements.txt installed first. Identify the project-code column in the metadata and subset rows matching the target codes — in this paper, 471_swedish and 514_imp_project as defined in Table S3. Then generate per-project boxplots of the recommended minimal metric set (Shannon for Information/entropy, Observed features for Richness, Berger-Parker for Dominance) to reproduce the Fig. 4 distributions, saving figures as PNG. Subsetting by metadata codes rather than file names is essential because the aggregated table merges all datasets into one table with metadata appended.

## Related tools

- **QIIME2** (Produced the per-dataset alpha diversity tables (via its diversity module) that are aggregated and then subsetted by project code) — https://qiime2.org/
- **anaconda** (Virtual environment manager used for the QIIME2 processing and notebook environment underlying the aggregated tables)
- **Jupyter** (Notebook environment where the aggregated table is loaded into pandas, subset by project codes, and plotted as boxplots)

## Examples

```
import pandas as pd; df = pd.read_csv('alpha_aggregated.tsv', sep='\t'); sub = df[df['project'].isin(['471_swedish', '514_imp_project'])]
```

## Evaluation signals

- Subsetted row counts match the number of samples belonging to 471_swedish and 514_imp_project in the metadata; no rows from other project codes remain
- Metric values in the subset are identical to those in alpha_aggregated.tsv / alpha-out_dataset-name.tsv (no re-calculation should occur)
- Resulting boxplot distributions for the two projects match the Fig. 4 panels (Shannon, Observed features, Berger-Parker)
- All 4,596 samples are present in the un-subsetted aggregated table before subsetting, confirming the merge did not drop samples

## Limitations

- Table S3 and Supplementary Material 1 are referenced but only a placeholder link is provided, so project-code definitions may need to be sourced from the repository's metadata files
- Raw sample data is not in the repository due to size; only accession numbers per dataset folder are available, so cohorts cannot be re-derived from raw reads without re-downloading from SRA
- The paper hypothesizes a subsampling artifact in the KarenThai cohort; per-project boxplots of cohorts with such artifacts may be misleading without larger sample sizes

## Evidence

- [readme] All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata.: "All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv. This data was later combined with each dataset's metadata."
- [discussion] utilizing data from two analyzed studies (projects 471_swedish and 514_imp_project, according to Table S3 codes): "utilizing data from two analyzed studies (projects 471_swedish and 514_imp_project, according to Table S3 codes)"
- [readme] The alpha diversity metrics for all samples and their metadata were later combined as alpha_aggregated.tsv.: "The alpha diversity metrics for all samples and their metadata were later combined as alpha_aggregated.tsv."
- [results] calculated for the sequence datasets obtained for a total of 4,596 stool samples included in 13 publicly available human microbiome projects: "calculated for the sequence datasets obtained for a total of 4,596 stool samples included in 13 publicly available human microbiome projects"
