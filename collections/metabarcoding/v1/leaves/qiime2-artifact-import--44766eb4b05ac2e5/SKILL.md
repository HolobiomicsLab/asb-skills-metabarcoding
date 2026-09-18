---
name: qiime2-artifact-import
description: Use when you hold a feature table (BIOM/TSV) or denoised count data that
  must become a QIIME 2 FeatureTable[Frequency] .qza artifact before running qiime
  diversity alpha or feature-table summarize/rarefy.
when_to_use_negative:
- Input is already a valid .qza FeatureTable artifact — skip import and proceed directly
  to summarize, rarefy, or diversity alpha
- Input is raw FASTQ needing denoising — that is a DADA2/DEBLUR denoising step, not
  a table import
- Goal is calculating alpha diversity metrics on the already-imported table — that
  is the downstream qiime diversity alpha step
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: QIIME2
  role: Provides `qiime tools import` to convert feature tables into .qza FeatureTable[Frequency]
    artifacts for downstream diversity analysis
  repo: https://qiime2.org/
- name: QIIME2 2022.2.0
  role: Specific QIIME 2 release in which the workflows and imports were run and validated
    as compatible
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment manager used to provision the QIIME 2 environment in which
    imports are executed
merged_aliases:
- qiime2-artifact-handling
merged_alias_records:
- alias: qiime2-artifact-handling
  slug: qiime2-artifact-handling
  cosine_score: 0.8367
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:5319e22f32c75f7fca8854977d762b03c2268a3c89db62fe6d5b85950d7f023a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - QIIME2 2022.2.0
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# qiime2-artifact-import

## Summary

Import external feature tables (e.g. the paper's negative-binomial synthetic tables or DEBLUR-denoised tables) into QIIME 2 as .qza FeatureTable artifacts inside an anaconda-provisioned QIIME 2 2022.2.0 environment, so that QIIME 2's diversity module can be run on them.

## When to use

Use when you hold a feature table (BIOM/TSV) or denoised count data that must become a QIIME 2 FeatureTable[Frequency] .qza artifact before running qiime diversity alpha or feature-table summarize/rarefy.

## When NOT to use

- Input is already a valid .qza FeatureTable artifact — skip import and proceed directly to summarize, rarefy, or diversity alpha
- Input is raw FASTQ needing denoising — that is a DADA2/DEBLUR denoising step, not a table import
- Goal is calculating alpha diversity metrics on the already-imported table — that is the downstream qiime diversity alpha step

## Inputs

- Feature table in an importable format (e.g. BIOM or TSV), such as the negative-binomial synthetic ASV tables (2x/10x/100x unevenness ratios)
- Denoised feature table produced with DEBLUR (QIIME 2 artifact or importable table)
- QIIME 2 2022.2.0 conda environment provisioned via anaconda

## Outputs

- QIIME 2 feature-table artifact (.qza) of type FeatureTable[Frequency]
- Feature-table summary confirming per-sample frequencies (via qiime feature-table summarize)

## How to apply

Provision QIIME 2 2022.2.0 (or a compatible newer version) in an anaconda virtual environment, as all sample processing in this study was done that way. Use `qiime tools import` with an explicit semantic type such as FeatureTable[Frequency] to convert each input table — e.g. the three negative-binomial synthetic feature tables with ASV2/ASV1 ratios of 2x, 10x, and 100x against an average of 0.8x, or the DEBLUR-denoised denoised table — into a .qza feature-table artifact. Decision point: this import step precedes diversity computation, and some metrics (ACE and Faith) cannot be recalculated on imported synthetic datasets due to technical limitations, so plan the downstream metric list at import time. After import, inspect the artifact with `qiime feature-table summarize` to confirm sample counts and sequencing-depth distribution before further steps (rarefaction or alpha diversity calculation with non-rarefied data).

## Related tools

- **QIIME2** (Provides `qiime tools import` to convert feature tables into .qza FeatureTable[Frequency] artifacts for downstream diversity analysis) — https://qiime2.org/
- **QIIME2 2022.2.0** (Specific QIIME 2 release in which the workflows and imports were run and validated as compatible) — https://qiime2.org/
- **anaconda** (Virtual environment manager used to provision the QIIME 2 environment in which imports are executed)

## Examples

```
qiime tools import --input-path synthetic_table.biom --type 'FeatureTable[Frequency]' --output-path synthetic_table.qza
```

## Evaluation signals

- `qiime tools peek`/validate on the output reports type FeatureTable[Frequency] and the artifact opens without error
- `qiime feature-table summarize` on the imported artifact reproduces the expected sample count and per-sample frequency totals from the source table
- For the rarefaction follow-up, every retained sample's total frequency equals the chosen sampling depth (value_in_range check on per-sample sums)
- The imported artifact supports the planned downstream step (qiime diversity alpha), except ACE and Faith which are known to fail on synthetic datasets
- Output artifact is saved with the study's naming convention (e.g. rarefied-feature-table.qza under the dataset folder)

## Limitations

- ACE and Faith metrics cannot be recalculated on synthetic datasets due to technical limitations, so imports of synthetic tables support only a subset of alpha metrics
- Raw sample data is not shared through the repository due to size; only accession numbers are available in each dataset's folder, so table provenance must be traced via SRA accessions
- Rarefied dataset availability is only described as 'data available in repository' without an explicit accession or location, so imported artifacts may not cover every derived dataset

## Evidence

- [methods] Import each of the three negative-binomial synthetic feature tables (ASV2/ASV1 ratios of 2x, 10x, and 100x against an average of 0.8x) into QIIME2 as feature-table artifacts.: [evidence span withheld — anchor status: not_found]
- [readme] All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [readme] Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code.: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
- [methods] Load the denoised feature table (produced with DEBLUR and stored as a QIIME 2 artifact) into the QIIME 2 2022.2.0 environment provisioned via anaconda.: [evidence span withheld — anchor status: not_found]
