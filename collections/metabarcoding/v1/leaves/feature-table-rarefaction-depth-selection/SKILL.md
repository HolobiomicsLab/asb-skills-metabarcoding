---
name: feature-table-rarefaction-depth-selection
description: Use when a denoised QIIME 2 feature table must be subsampled without
  replacement to an identical per-sample depth to validate alpha diversity results,
  or when uneven sequencing depth could confound metric comparisons.
when_to_use_negative:
- 'Primary metric calculations: the paper calculated alpha diversity metrics with
  non-rarefied data to preserve as much information as possible — rarefy only for
  the validation pass.'
- Input is already rarefied or per-sample counts are already uniform; re-rarefying
  discards additional reads unnecessarily.
- The chosen depth would drop so many samples that an adequate number of samples per
  dataset is no longer retained.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: QIIME2
  role: Provides 'qiime feature-table rarefy' and 'qiime feature-table summarize'
    for subsampling to a fixed depth and verifying per-sample totals
  repo: https://qiime2.org/
- name: QIIME2 2022.2.0
  role: Specific QIIME 2 version in which the rarefaction workflows were run
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment used to provision the QIIME 2 2022.2.0 installation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:7a8dbecf67e41dbc37abdc40d445fd80629bd683735ab59390ddb2d6eaa79b16
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# feature-table-rarefaction-depth-selection

## Summary

Rarefies a denoised QIIME 2 feature table (e.g. DEBLUR output) to a fixed, chosen sequencing depth so that every retained sample has an identical total frequency, allowing alpha diversity results computed on non-rarefied data to be validated on rarefied datasets.

## When to use

Use when a denoised QIIME 2 feature table must be subsampled without replacement to an identical per-sample depth to validate alpha diversity results, or when uneven sequencing depth could confound metric comparisons.

## When NOT to use

- Primary metric calculations: the paper calculated alpha diversity metrics with non-rarefied data to preserve as much information as possible — rarefy only for the validation pass.
- Input is already rarefied or per-sample counts are already uniform; re-rarefying discards additional reads unnecessarily.
- The chosen depth would drop so many samples that an adequate number of samples per dataset is no longer retained.

## Inputs

- Denoised QIIME 2 feature table artifact (.qza), e.g. produced with DEBLUR
- Chosen even sampling-depth value selected after inspecting the depth distribution

## Outputs

- rarefied-feature-table.qza — a valid QIIME 2 feature-table artifact with every retained sample at the chosen sampling depth

## How to apply

Load the denoised feature table (produced with DEBLUR and stored as a .qza artifact) into a QIIME 2 2022.2.0 environment provisioned via anaconda. Inspect the per-sample sequencing-depth distribution with 'qiime feature-table summarize' and choose an even sampling depth that retains an adequate number of samples per dataset — this is the key decision point, trading retained samples against retained reads. Apply 'qiime feature-table rarefy' with the chosen --p-sampling-depth so each sample is subsampled without replacement to identical depth, dropping samples below the threshold. Confirm the rarefied table's per-sample counts with 'qiime feature-table summarize' and save it as rarefied-feature-table.qza. The paper's rationale is that primary alpha diversity metrics were calculated on non-rarefied data to preserve as much information as possible, with rarefied datasets used only as an independent validation of the reported results.

## Related tools

- **QIIME2** (Provides 'qiime feature-table rarefy' and 'qiime feature-table summarize' for subsampling to a fixed depth and verifying per-sample totals) — https://qiime2.org/
- **QIIME2 2022.2.0** (Specific QIIME 2 version in which the rarefaction workflows were run) — https://qiime2.org/
- **anaconda** (Virtual environment used to provision the QIIME 2 2022.2.0 installation)

## Examples

```
qiime feature-table rarefy --i-table feature-table.qza --p-sampling-depth 1000 --o-rarefied-table rarefied-feature-table.qza
```

## Evaluation signals

- The output artifact rarefied-feature-table.qza exists and is a valid QIIME 2 feature-table artifact.
- Every retained sample's total frequency equals the chosen sampling depth (value_in_range check on per-sample sums).
- Samples whose original sequencing depth was below the chosen threshold are absent from the rarefied table.
- Alpha diversity metrics recomputed on the rarefied table are consistent with (validate) the reported non-rarefied results.

## Limitations

- Rarefied dataset availability is only described as 'data available in repository' without an explicit accession or location, making reproducibility of the validation pass harder.
- Rarefaction discards reads and samples below the threshold; the paper's primary results therefore used non-rarefied data, and subsampling artifacts (e.g. the hypothesized artifact in the KarenThai cohort) can still bias depth-driven comparisons.
- Commands were run on QIIME 2 2022.2.0; the README notes they 'should be compatible with newer versions', but compatibility is not guaranteed.

## Evidence

- [results] Alpha diversity metrics calculated with non-rarefied data; results also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [intro] QIIME 2 version and environment: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] Workflow version compatibility: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [discussion] Rarefied dataset availability caveat: "following standard procedures to validate our claims (data available in repository), but all tests were performed with non-rarefied data."
