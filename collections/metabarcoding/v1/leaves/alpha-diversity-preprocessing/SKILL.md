---
name: alpha-diversity-preprocessing
description: Use when a denoised QIIME 2 feature-table artifact (.qza) must be subsampled
  to identical per-sample sequencing depth — e.g. to validate alpha diversity metrics
  that were primarily computed on non-rarefied data.
when_to_use_negative:
- Input is raw reads or an undenoised table — denoising (DEBLUR/DADA2) and QIIME2
  best-practices processing must precede rarefaction
- 'Primary alpha diversity calculations: the paper computes metrics on non-rarefied
  data to preserve information; rarefied tables serve only as validation'
- Samples whose depths fall below the chosen sampling depth should be expected to
  be dropped, not padded or partially analyzed
edam_operation: http://edamontology.org/operation_3235
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: QIIME2
  role: feature-table rarefy and feature-table summarize commands perform the depth
    selection, subsampling, and verification
  repo: https://qiime2.org/
- name: QIIME2 2022.2.0
  role: specific suite version the workflows were run on, installed in the analysis
    environment
  repo: https://qiime2.org/
- name: anaconda
  role: virtual environment in which all sample processing with QIIME2 was done
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:f0d2edf1e6c091b37dd4a8e8583485b13c3581d72f7e03f6c16f3cb13963c6c2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: http://edamontology.org/operation_3235
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# alpha-diversity-preprocessing

## Summary

Rarefaction-based preprocessing of denoised QIIME 2 feature tables (DEBLUR output) at a fixed sequencing depth, used to validate alpha diversity results obtained from non-rarefied data in this 4,596-stool-sample, 13-project microbiome analysis.

## When to use

Use when a denoised QIIME 2 feature-table artifact (.qza) must be subsampled to identical per-sample sequencing depth — e.g. to validate alpha diversity metrics that were primarily computed on non-rarefied data.

## When NOT to use

- Input is raw reads or an undenoised table — denoising (DEBLUR/DADA2) and QIIME2 best-practices processing must precede rarefaction
- Primary alpha diversity calculations: the paper computes metrics on non-rarefied data to preserve information; rarefied tables serve only as validation
- Samples whose depths fall below the chosen sampling depth should be expected to be dropped, not padded or partially analyzed

## Inputs

- Denoised feature table as a QIIME 2 artifact (.qza), produced with DEBLUR
- Chosen even sampling-depth value (integer read count) selected from the sample depth distribution

## Outputs

- rarefied-feature-table.qza — a valid QIIME 2 feature-table artifact with identical per-sample total frequencies
- Per-sample frequency summary confirming each retained sample equals the chosen sampling depth

## How to apply

Load the DEBLUR-denoised feature table (a QIIME 2 artifact) into the QIIME2 2022.2.0 environment provisioned via anaconda and inspect the sample depth distribution with qiime feature-table summarize to choose an even sampling depth that retains an adequate number of samples per dataset. Apply qiime feature-table rarefy with the chosen --p-sampling-depth to subsample each sample without replacement to identical depth, dropping samples below the threshold. Re-run qiime feature-table summarize on the output to confirm per-sample counts and save it as rarefied-feature-table.qza. Rationale: the paper computes alpha diversity metrics primarily with non-rarefied data 'to preserve as much information as possible', and uses these rarefied tables only as a validation of the reported results; per-sample totals must equal exactly the chosen sampling depth.

## Related tools

- **QIIME2** (feature-table rarefy and feature-table summarize commands perform the depth selection, subsampling, and verification) — https://qiime2.org/
- **QIIME2 2022.2.0** (specific suite version the workflows were run on, installed in the analysis environment) — https://qiime2.org/
- **anaconda** (virtual environment in which all sample processing with QIIME2 was done)

## Examples

```
qiime feature-table rarefy --i-table feature-table.qza --p-sampling-depth <depth> --o-rarefied-table rarefied-feature-table.qza
```

## Evaluation signals

- Output artifact rarefied-feature-table.qza exists and is recognized as a valid QIIME 2 feature-table artifact
- Per-sample total frequency equals the chosen sampling depth for every retained sample (value_in_range/equality check via qiime feature-table summarize)
- Samples below the chosen sampling depth are absent from the rarefied table
- Alpha diversity metrics recomputed on the rarefied table are consistent with (validate) the primary non-rarefied results

## Limitations

- All primary tests were performed with non-rarefied data; rarefied dataset availability is only described as 'data available in repository' without explicit accession or location
- Choosing a sampling depth that is too high drops many samples per dataset; the depth must be tuned to retain an adequate number of samples
- Rarefied results are a validation layer only — they do not replace the non-rarefied primary calculations

## Evidence

- [results] Alpha diversity metrics calculated with non-rarefied data; results also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [readme] QIIME2 on anaconda environment: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] Workflow version: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [discussion] Non-rarefied primary analysis caveat: "following standard procedures to validate our claims (data available in repository), but all tests were performed with non-rarefied data."
