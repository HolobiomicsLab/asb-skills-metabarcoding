---
name: conda-environment-reproducible-setup
description: Use when reproducing or extending the paper's Jupyter-notebook analyses
  of alpha diversity metrics, or when running QIIME2 diversity-module commands on
  the rarefied/non-rarefied tables requires a version-pinned environment.
when_to_use_negative:
- A working QIIME2/conda environment with matching packages already exists and only
  analyses need re-running
- The task is only downloading raw reads from SRA with sra-tools and does not involve
  notebook or QIIME2 execution
- The target analysis requires tool versions different from QIIME 2022.2.0 without
  first confirming metric behavior is compatible
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: anaconda
  role: Hosts the conda virtual environment in which QIIME2 and all sample processing
    were run
- name: QIIME2 2022.2.0
  role: Version-pinned tool suite installed inside the environment; its diversity
    module computes the 19 alpha diversity metrics
  repo: https://qiime2.org/
- name: Jupyter
  role: Notebook runtime whose kernel must be pointed at the created environment for
    data exploration and visualization
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:01f17f779a097d81ada58ba3219d4a5d67a24594f52f19c36ae849fcd6ce6ee2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# conda-environment-reproducible-setup

## Summary

Set up a reproducible analysis environment (conda or Python virtualenv) pinned to the exact tool versions used by the alpha-diversity study (QIIME 2022.2.0), installing notebook dependencies from requirements.txt so all 19 alpha diversity metrics can be recomputed identically.

## When to use

Use when reproducing or extending the paper's Jupyter-notebook analyses of alpha diversity metrics, or when running QIIME2 diversity-module commands on the rarefied/non-rarefied tables requires a version-pinned environment.

## When NOT to use

- A working QIIME2/conda environment with matching packages already exists and only analyses need re-running
- The task is only downloading raw reads from SRA with sra-tools and does not involve notebook or QIIME2 execution
- The target analysis requires tool versions different from QIIME 2022.2.0 without first confirming metric behavior is compatible

## Inputs

- requirements.txt from the mauroibz/alpha-diversities repository
- QIIME2 2022.2.0 release (via anaconda/conda)
- Jupyter notebooks in the repository

## Outputs

- Activated conda or Python venv with QIIME2 2022.2.0 and notebook dependencies installed
- Jupyter kernel configured to that environment for running the alpha-diversity notebooks

## How to apply

Build an anaconda (conda) virtual environment carrying QIIME 2022.2.0, or alternatively a plain Python virtualenv, since the article's workflows were 'ran on qiime 2022.2.0, but should be compatible with newer versions of the code'. Install the notebook package requirements with `pip install -r requirements.txt` from the mauroibz/alpha-diversities repository, then select that environment as the Jupyter kernel and run each notebook cell sequentially. All sample processing and alpha diversity calculation (QIIME2's diversity module) must happen inside this single environment so that DADA2/DEBLUR outputs and the alpha-out_dataset-name.tsv aggregations are comparable across the 13 stool-sample projects. Choose a newer QIIME2 release only if you verify metric behavior is unchanged, since denoiser versions are a known source of variability in estimated richness.

## Related tools

- **anaconda** (Hosts the conda virtual environment in which QIIME2 and all sample processing were run)
- **QIIME2 2022.2.0** (Version-pinned tool suite installed inside the environment; its diversity module computes the 19 alpha diversity metrics) — https://qiime2.org/
- **Jupyter** (Notebook runtime whose kernel must be pointed at the created environment for data exploration and visualization)

## Examples

```
source venv/bin/activate && pip install -r requirements.txt
```

## Evaluation signals

- `qiime --version` (or equivalent) inside the activated environment reports 2022.2.0 or a confirmed-compatible newer release
- `pip install -r requirements.txt` completes without dependency conflicts and all notebook cells run sequentially without import errors
- Recomputed alpha metrics from QIIME2's diversity module match the values in the repository's alpha-out_dataset-name.tsv files for the same datasets
- Rarefied and non-rarefied recomputations of the 19 metrics agree with the paper's four categories (Richness, Dominance, Phylogenetics, Information)

## Limitations

- Raw sample data are not shared in the repository due to size; only accession numbers (SraRunTable.txt per dataset folder) are provided, so environment setup alone does not reproduce data acquisition
- No version numbers were reported for DADA2 or DEBLUR, so the environment cannot fully pin the denoising step
- The article notes workflows should be compatible with newer QIIME2 versions, but compatibility of metric behavior under newer releases is not formally guaranteed

## Evidence

- [intro] All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [intro] Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code.: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [readme] Install requirments > pip install -r requirements.txt: "Install requirments
    > pip install -r requirements.txt"
- [readme] Select your virtual enviroment as your kernel for the notebooks and run each cell in sequential order.: "Select your virtual enviroment as your kernel for the notebooks and run each cell in sequential order."
