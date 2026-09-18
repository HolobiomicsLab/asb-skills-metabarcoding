---
name: microbiome-count-normalization
description: Use when a DEBLUR/DADA2-denoised QIIME 2 feature table has uneven sequencing
  depths across samples and you need rarefied data to validate alpha diversity metrics
  calculated on non-rarefied data.
when_to_use_negative:
- 'You are computing the primary alpha diversity values: this paper calculates metrics
  on non-rarefied data to preserve as much information as possible, and uses rarefaction
  only for validation'
- The input is already a rarefied feature table or all samples already share the same
  total frequency
- Too many samples fall below the candidate sampling depth, so rarefying would drop
  samples the downstream comparison requires to retain
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME2
  role: Provides the feature-table rarefy and feature-table summarize commands used
    to rarefy and verify per-sample counts
  repo: https://qiime2.org/
- name: QIIME2 2022.2.0
  role: Pinned QIIME 2 version in which the rarefaction workflow was run (compatible
    with newer versions)
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment in which the QIIME 2 processing was performed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:925d5a8c3559c44907977082d72c4b70783f280f0f285abd09a56649af1e9972
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 4
---

# microbiome-count-normalization

## Summary

Normalizes a denoised QIIME 2 feature table to a common sequencing depth by rarefying (subsampling without replacement) each sample, so per-sample totals are identical and alpha diversity results can be validated against non-rarefied calculations as done in this paper's pipeline.

## When to use

Use when a DEBLUR/DADA2-denoised QIIME 2 feature table has uneven sequencing depths across samples and you need rarefied data to validate alpha diversity metrics calculated on non-rarefied data.

## When NOT to use

- You are computing the primary alpha diversity values: this paper calculates metrics on non-rarefied data to preserve as much information as possible, and uses rarefaction only for validation
- The input is already a rarefied feature table or all samples already share the same total frequency
- Too many samples fall below the candidate sampling depth, so rarefying would drop samples the downstream comparison requires to retain

## Inputs

- Denoised QIIME 2 feature-table artifact (.qza, DEBLUR output)
- Chosen even sampling depth (reads per sample, selected from the depth distribution)

## Outputs

- rarefied-feature-table.qza (QIIME 2 feature-table artifact with identical per-sample total frequencies)

## How to apply

Load the denoised feature table (produced with DEBLUR and stored as a .qza artifact) into the QIIME 2 2022.2.0 environment provisioned via anaconda. Inspect the per-sample sequencing-depth distribution (e.g. qiime feature-table summarize) to select an even sampling depth that retains an adequate number of samples per dataset — this is the key decision point, since deeper depths retain more information but drop more low-depth samples. Apply qiime feature-table rarefy at the chosen --p-sampling-depth so each sample is subsampled without replacement to identical depth and samples below the threshold are dropped. Confirm the rarefied table with qiime feature-table summarize and save it as rarefied-feature-table.qza. Rationale: in this study alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible, and rarefied datasets were used as a secondary validation of all reported results.

## Related tools

- **QIIME2** (Provides the feature-table rarefy and feature-table summarize commands used to rarefy and verify per-sample counts) — https://qiime2.org/
- **QIIME2 2022.2.0** (Pinned QIIME 2 version in which the rarefaction workflow was run (compatible with newer versions)) — https://qiime2.org/
- **anaconda** (Virtual environment in which the QIIME 2 processing was performed)

## Examples

```
qiime feature-table rarefy --i-table denoised-feature-table.qza --p-sampling-depth <chosen_depth> --o-rarefied-table rarefied-feature-table.qza
```

## Evaluation signals

- Output artifact rarefied-feature-table.qza exists and is a valid QIIME 2 feature-table artifact
- Every retained sample's total frequency equals the chosen sampling depth (value_in_range check on per-sample sums)
- Samples whose depth was below the chosen sampling depth are absent from the rarefied table
- qiime feature-table summarize on the rarefied table reports a single common depth across all samples
- Alpha diversity metrics recomputed on the rarefied table agree with (validate) the primary non-rarefied results

## Limitations

- Rarefying discards reads and drops samples below the chosen depth, which is why the paper's primary alpha diversity calculations used non-rarefied data
- Rarefied dataset availability is only described as 'data available in repository' without an explicit accession or location, so reproducing the exact rarefied tables may require regenerating them
- Choice of sampling depth is dataset-specific (retention of adequate samples per dataset vs. information retained); no universal threshold is provided

## Evidence

- [results] Alpha diversity metrics calculated with non-rarefied data; results also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [discussion] Rarefied validation described in discussion: "following standard procedures to validate our claims (data available in repository), but all tests were performed with non-rarefied data."
- [readme] QIIME2 on anaconda environment: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] QIIME 2 version used for workflows: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
