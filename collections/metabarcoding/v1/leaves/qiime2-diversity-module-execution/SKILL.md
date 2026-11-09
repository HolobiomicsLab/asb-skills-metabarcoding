---
name: qiime2-diversity-module-execution
description: Use when alpha diversity metrics (richness, dominance, phylogenetic,
  information categories) must be recalculated from QIIME2-processed feature tables,
  e.g. to reproduce or validate metric behavior on rarefied vs non-rarefied stool-sample
  datasets.
when_to_use_negative:
- When the goal is beta diversity or sample-sample comparison rather than the 19 within-sample
  alpha diversity metrics
- When raw sequence processing/denoising (DADA2/DEBLUR) has not yet been run — this
  skill consumes QIIME2 feature tables, not raw FASTQ
- When rarefaction-only results are needed exclusively; the paper's primary calculations
  intentionally use non-rarefied data to preserve information
edam_topics:
- http://edamontology.org/topic_3316
tools:
- name: QIIME2
  role: Suite used for all sample processing; its diversity module computes all alpha
    diversity metrics
  repo: https://qiime2.org/
- name: QIIME2 2022.2.0
  role: Specific QIIME2 release on which the alpha diversity workflows were run
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment platform hosting the QIIME2 installation used to run the
    diversity module
- name: Jupyter
  role: Notebooks used to explore aggregated alpha diversity data and build visualizations
    from the module's outputs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:6d396c4e63986713cb1b2ef3e4487a6d8c29468b505898a93c70c388d034b683
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - QIIME2
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# qiime2-diversity-module-execution

## Summary

Recompute microbial alpha diversity metrics with QIIME2's diversity module on non-rarefied feature tables (validated against rarefied tables) and aggregate the per-sample values into dataset-level tables such as alpha-out_dataset-name.tsv.

## When to use

Use when alpha diversity metrics (richness, dominance, phylogenetic, information categories) must be recalculated from QIIME2-processed feature tables, e.g. to reproduce or validate metric behavior on rarefied vs non-rarefied stool-sample datasets.

## When NOT to use

- When the goal is beta diversity or sample-sample comparison rather than the 19 within-sample alpha diversity metrics
- When raw sequence processing/denoising (DADA2/DEBLUR) has not yet been run — this skill consumes QIIME2 feature tables, not raw FASTQ
- When rarefaction-only results are needed exclusively; the paper's primary calculations intentionally use non-rarefied data to preserve information

## Inputs

- QIIME2 feature table (.qza) per dataset, generated with DADA2 and DEBLUR denoising (downstream analyses use DEBLUR data)
- Rarefied dataset tables for validation runs
- Per-dataset sample metadata from SRA accession files (e.g. SraRunTable.txt)

## Outputs

- Per-dataset alpha diversity table: datasets/dataset-name/alpha-out_dataset-name.tsv
- Combined alpha_aggregated.tsv of all sample metrics plus metadata
- Rarefied vs non-rarefied comparison of the 19 metrics per dataset

## How to apply

Set up a QIIME2 2022.2.0 environment inside anaconda (workflows ran on that version but should be compatible with newer releases) and load the feature tables built by the paper's DADA2/DEBLUR processing pipeline. Compute all 19 alpha diversity metrics using QIIME2's diversity module; the paper calculates them on non-rarefied data to preserve as much information as possible, then additionally validates all results on rarefied datasets. For each dataset, aggregate the per-sample metric values into a table named datasets/dataset-name/alpha-out_dataset-name.tsv and combine it with that dataset's metadata (later merged into alpha_aggregated.tsv). Align rarefied and non-rarefied metric values per dataset and compare metric-by-metric across the 13 stool-sample projects, confirming that conclusions for the four categories (Richness, Dominance, Phylogenetics, Information) are unchanged. Follow QIIME2 best practices as documented in the repo's pipeline_scripts/workflow.txt template.

## Related tools

- **QIIME2** (Suite used for all sample processing; its diversity module computes all alpha diversity metrics) — https://qiime2.org/
- **QIIME2 2022.2.0** (Specific QIIME2 release on which the alpha diversity workflows were run) — https://qiime2.org/
- **anaconda** (Virtual environment platform hosting the QIIME2 installation used to run the diversity module)
- **Jupyter** (Notebooks used to explore aggregated alpha diversity data and build visualizations from the module's outputs)

## Examples

```
qiime diversity alpha --i-table deblur_table.qza --p-metric shannon --o-alpha-diversity shannon_vector.qza
```

## Evaluation signals

- All 19 alpha diversity metrics appear in each alpha-out_dataset-name.tsv and their values reproduce when recomputed via QIIME2's diversity module
- Rarefied recomputations align with non-rarefied results per dataset, confirming the four metric categories (Richness, Dominance, Phylogenetics, Information) behave identically
- Aggregated table covers all 4,596 stool samples from the 13 publicly available human microbiome projects
- Merged alpha_aggregated.tsv correctly joins metric values with each dataset's metadata
- Environment resolves with the pinned QIIME2 2022.2.0 (or compatible newer version) and requirements.txt dependencies

## Limitations

- Raw sample data is not shared in the repository due to size; only SRA accession numbers (SraRunTable.txt per dataset folder) are provided
- ACE and Faith could not be recalculated on the synthetic validation datasets due to technical limitations
- Faith's metric is algorithmic rather than formula-based and is 'continuously suffering changes', so its formula is not documented
- Rarefied dataset availability is described only as 'data available in repository' without an explicit accession or location

## Evidence

- [readme] All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv.: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv"
- [intro] All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [intro] Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code.: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [results] alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [results] Alpha metrics used in 68 microbiome studies or in the QIIME 2 suite were grouped into four categories: "Alpha metrics used in 68 microbiome studies or in the QIIME 2 suite were grouped into four categories"
- [discussion] we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations.: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
