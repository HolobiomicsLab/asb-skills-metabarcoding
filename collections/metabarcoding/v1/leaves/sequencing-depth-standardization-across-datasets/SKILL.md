---
name: sequencing-depth-standardization-across-datasets
description: Use when a denoised QIIME 2 feature table (.qza) has highly variable
  per-sample read counts and alpha diversity results need to be validated on rarefied
  data at a single even sampling depth.
when_to_use_negative:
- Alpha diversity metrics are computed on the primary non-rarefied data — rarefaction
  here is only for validation, not the main analysis
- Input is not a denoised feature table (e.g. raw reads or FASTQ), which must first
  be processed via QIIME2 denoising (DADA2/DEBLUR)
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: QIIME2 2022.2.0
  role: Provides the feature-table rarefy and feature-table summarize commands used
    to subsample each sample to the chosen sampling depth and verify per-sample frequencies
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment in which the QIIME2 2022.2.0 workflow is provisioned
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:22b7e487c367a65c9a0a854bd1433a9f1bf5861a66e2d8cb870ce02f964c0f97
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# sequencing-depth-standardization-across-datasets

## Summary

Standardize sequencing depth across samples by rarefying a QIIME 2 denoised feature table (e.g. DEBLUR output) to a fixed sampling depth so alpha diversity results can be validated against rarefied data, complementing the paper's primary non-rarefied calculations.

## When to use

Use when a denoised QIIME 2 feature table (.qza) has highly variable per-sample read counts and alpha diversity results need to be validated on rarefied data at a single even sampling depth.

## When NOT to use

- Alpha diversity metrics are computed on the primary non-rarefied data — rarefaction here is only for validation, not the main analysis
- Input is not a denoised feature table (e.g. raw reads or FASTQ), which must first be processed via QIIME2 denoising (DADA2/DEBLUR)

## Inputs

- Denoised QIIME 2 feature-table artifact (.qza) produced with DEBLUR
- Per-sample sequencing-depth distribution (e.g. from qiime feature-table summarize)

## Outputs

- rarefied-feature-table.qza (QIIME 2 feature-table artifact with every retained sample at identical total frequency)

## How to apply

Load the denoised feature table (produced with DEBLUR and stored as a QIIME 2 2022.2.0 artifact in an anaconda environment) and inspect the sequencing-depth distribution with 'qiime feature-table summarize' to pick a sampling depth that retains an adequate number of samples per dataset. Apply 'qiime feature-table rarefy' at the chosen --p-sampling-depth to subsample each sample without replacement to identical depth, dropping samples below the threshold. Confirm the rarefied table's per-sample counts with 'qiime feature-table summarize' and save it as rarefied-feature-table.qza. Rationale: the paper computes alpha diversity on non-rarefied data to preserve information, and uses rarefied datasets only as a validation step.

## Related tools

- **QIIME2 2022.2.0** (Provides the feature-table rarefy and feature-table summarize commands used to subsample each sample to the chosen sampling depth and verify per-sample frequencies) — https://qiime2.org/
- **anaconda** (Virtual environment in which the QIIME2 2022.2.0 workflow is provisioned)

## Examples

```
qiime feature-table rarefy --i-table feature-table.qza --p-sampling-depth <chosen-depth> --o-rarefied-table rarefied-feature-table.qza
```

## Evaluation signals

- The output artifact exists and is a valid QIIME 2 feature-table artifact
- Every retained sample's total frequency equals the chosen sampling depth (value_in_range check on per-sample sums)
- Samples below the chosen sampling depth are absent from the rarefied table
- The rarefied table's per-sample counts are confirmed via qiime feature-table summarize

## Limitations

- Rarefied dataset availability is only described as 'data available in repository' without explicit accession or location
- The primary tests in the paper were performed with non-rarefied data; rarefied tables are a validation aid, and dropping samples below the depth reduces the cohort
- QIIME 2, DADA2 and DEBLUR version numbers are incompletely reported (workflow ran on QIIME 2022.2.0), so exact reproducibility across versions is not guaranteed

## Evidence

- [results] alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [methods] Apply qiime feature-table rarefy at the chosen sampling-depth parameter to subsample each sample without replacement to identical depth, dropping samples below the threshold.: [evidence span withheld — anchor status: not_found]
- [methods] verify the output artifact exists, is a valid QIIME 2 feature-table artifact, and that every retained sample's total frequency equals the chosen sampling depth: [evidence span withheld — anchor status: not_found]
- [readme] All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code.: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
