---
name: python-virtual-environment-reproducibility
description: Use when reproducing a repository's Jupyter-notebook-based analysis (e.g.
  the mauroibz/alpha-diversities alpha diversity exploration) and you need an isolated
  environment with pinned dependencies before running cells.
when_to_use_negative:
- Reproducing the QIIME2 sample-processing stage (DADA2/DEBLUR denoising) — that workflow
  runs on QIIME2 2022.2.0 in an anaconda environment, not this Python venv
- The repository does not contain raw sample data (only accession numbers like SraRunTable.txt
  are provided), so the venv cannot be used to regenerate raw data from scratch
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: anaconda
  role: Virtual environment platform used for the QIIME2 sample-processing stage alongside
    the Python venv used for notebooks
- name: QIIME2
  role: External tool whose outputs (alpha diversity metrics per dataset) are consumed
    by the notebooks running inside this environment
  repo: https://qiime2.org/
- name: mauroibz/alpha-diversities
  role: Source repository providing requirements.txt, Jupyter notebooks, and dataset
    tables
  repo: https://github.com/mauroibz/alpha-diversities
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:cfbba2295f646a65184a0e6b6c0bbe0815a9e203061809c5d41a0cf57d25b642
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - anaconda
  - mauroibz/alpha-diversities
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 5
---

# python-virtual-environment-reproducibility

## Summary

Set up an isolated, pinned Python virtual environment from a repository's requirements.txt so that the repository's Jupyter notebook analyses (here, the alpha-diversities study of 19 microbial alpha diversity metrics) run reproducibly and sequentially.

## When to use

Use when reproducing a repository's Jupyter-notebook-based analysis (e.g. the mauroibz/alpha-diversities alpha diversity exploration) and you need an isolated environment with pinned dependencies before running cells.

## When NOT to use

- Reproducing the QIIME2 sample-processing stage (DADA2/DEBLUR denoising) — that workflow runs on QIIME2 2022.2.0 in an anaconda environment, not this Python venv
- The repository does not contain raw sample data (only accession numbers like SraRunTable.txt are provided), so the venv cannot be used to regenerate raw data from scratch

## Inputs

- mauroibz/alpha-diversities repository contents (Jupyter notebooks, requirements.txt, datasets/ directory)
- alpha-out_dataset-name.tsv metrics tables and dataset metadata to be combined in the notebooks

## Outputs

- Activated Python virtual environment with pinned dependencies from requirements.txt
- Executed Jupyter notebooks reproducing the alpha diversity exploration and visualizations

## How to apply

Clone the alpha-diversities repository (Zenodo DOI 10.5281/zenodo.8170289) and build a Python virtual environment with `python -m venv venv/`, then activate it with `source venv/bin/activate` and install pinned dependencies via `pip install -r requirements.txt`. Note that sample processing itself is done separately with QIIME2 2022.2.0 on an anaconda virtual environment, so the venv covers only the notebook-based data exploration and visualization layer. Register the virtual environment as the Jupyter kernel and run each notebook cell in sequential order, as the notebooks build up state (e.g. aggregated alpha diversity tables like alpha-out_dataset-name.tsv and alpha_aggregated.tsv) that later cells depend on. Steps may vary by platform; the README points to the Python venv documentation for platform-specific variations.

## Related tools

- **anaconda** (Virtual environment platform used for the QIIME2 sample-processing stage alongside the Python venv used for notebooks)
- **QIIME2** (External tool whose outputs (alpha diversity metrics per dataset) are consumed by the notebooks running inside this environment) — https://qiime2.org/
- **mauroibz/alpha-diversities** (Source repository providing requirements.txt, Jupyter notebooks, and dataset tables) — https://github.com/mauroibz/alpha-diversities

## Examples

```
python -m venv venv/; source venv/bin/activate; pip install -r requirements.txt
```

## Evaluation signals

- `pip install -r requirements.txt` completes inside the venv without dependency resolution errors
- The venv appears as a selectable Jupyter kernel and the first notebook cell runs
- All notebook cells execute in sequential order without NameError/state failures from out-of-order execution
- Resulting tables (e.g. alpha-out_dataset-name.tsv, alpha_aggregated.tsv) are regenerated matching the repository's described structure

## Limitations

- Raw sample data is not shared in the repository due to size; only accession numbers are available, so the environment setup alone cannot reproduce raw data acquisition (SRA downloads via sra-tools are a separate step)
- Notebook steps may vary depending on the practitioner's platform, per the README's pointer to the Python venv documentation
- QIIME2 workflows were run on qiime 2022.2.0 and compatibility with newer versions is asserted but not guaranteed

## Evidence

- [readme] Exploration of data and vizualizations were carried out on different jupyter notebooks. Package requirements are on requirements.txt.: "Exploration of data and vizualizations were carried out on different jupyter notebooks. Package requirements are on requirements.txt."
- [readme] Build a python virtual enviroment > python -m venv venv/ ... Install requirments > pip install -r requirements.txt: "Build a python virtual enviroment
    > python -m venv venv/

    Activate the python virtual enviroment
    > source venv/bin/activate

    Install requirments
    > pip install -r requirements.txt"
- [readme] Select your virtual enviroment as your kernel for the notebooks and run each cell in sequential order.: "Select your virtual enviroment as your kernel for the notebooks and run each cell in sequential order."
- [intro] All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
