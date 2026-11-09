---
name: non-rarefied-diversity-calculation
description: Use when you have a QIIME 2 feature-table artifact (or a synthetic negative-binomial
  table) and need alpha diversity metric values computed without rarefaction depth
  loss, e.g. for metric sensitivity comparisons.
when_to_use_negative:
- When the goal is to recalculate ACE or Faith on synthetic datasets — these metrics
  could not be recalculated due to technical limitations
- When the analysis question requires rarefaction-based normalization as the primary
  treatment rather than non-rarefied calculation with rarefied validation
- When the input is not a QIIME 2 feature table (e.g. raw FASTQ reads still needing
  DADA2/DEBLUR denoising first)
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: diversity module (qiime diversity alpha) computes all alpha diversity metrics
    on the non-rarefied feature tables
  repo: https://qiime2.org/
- name: anaconda
  role: virtual environment hosting the QIIME 2 installation used for sample processing
    and metric calculation
- name: sra-tools
  role: downloads the source datasets from SRA before feature tables are built (upstream
    of this skill)
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:04e469cf893ee4638297927f733709817e60b9667d100d6addb2d51ff26d552d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - anaconda
  - sra-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# non-rarefied-diversity-calculation

## Summary

Compute QIIME 2 alpha diversity metrics on non-rarefied feature tables to preserve as much information as possible, then aggregate all metrics per dataset into alpha-out_<dataset-name>.tsv tables, optionally cross-validating against rarefied data.

## When to use

Use when you have a QIIME 2 feature-table artifact (or a synthetic negative-binomial table) and need alpha diversity metric values computed without rarefaction depth loss, e.g. for metric sensitivity comparisons.

## When NOT to use

- When the goal is to recalculate ACE or Faith on synthetic datasets — these metrics could not be recalculated due to technical limitations
- When the analysis question requires rarefaction-based normalization as the primary treatment rather than non-rarefied calculation with rarefied validation
- When the input is not a QIIME 2 feature table (e.g. raw FASTQ reads still needing DADA2/DEBLUR denoising first)

## Inputs

- QIIME 2 feature-table artifact (.qza), non-rarefied (real or synthetic, e.g. negative-binomial synthetic tables with ASV2/ASV1 ratios of 2x, 10x, 100x)
- Dataset name used to name the output file alpha-out_<dataset-name>.tsv
- Python virtual environment with dependencies pinned in requirements.txt (for downstream notebooks)

## Outputs

- alpha-out_<dataset-name>.tsv: per-sample values for all available QIIME 2 alpha metrics (except ACE and Faith on synthetic datasets)
- alpha_aggregated.tsv: metrics combined across datasets with sample metadata
- Optional rarefied-data recomputation used only as validation

## How to apply

Import each feature table into QIIME 2 (2022.2.0 in the reference workflow, on an anaconda environment) and run the diversity module (qiime diversity alpha) on non-rarefied data, computing every available alpha metric except ACE and Faith, which cannot be recalculated on synthetic datasets due to technical limitations. Choose non-rarefaction deliberately: the study's rationale is to preserve as much information as possible when calculating metrics. Export the metric values and aggregate them per dataset into a table named alpha-out_<dataset-name>.tsv (e.g. for the three synthetic negative-binomial tables with ASV2/ASV1 ratios of 2x, 10x, and 100x against an average of 0.8x). Finally, validate the claim by also recomputing metrics on rarefied versions of the datasets and checking that conclusions hold under both treatments.

## Related tools

- **QIIME2** (diversity module (qiime diversity alpha) computes all alpha diversity metrics on the non-rarefied feature tables) — https://qiime2.org/
- **anaconda** (virtual environment hosting the QIIME 2 installation used for sample processing and metric calculation)
- **sra-tools** (downloads the source datasets from SRA before feature tables are built (upstream of this skill)) — https://github.com/ncbi/sra-tools

## Examples

```
qiime diversity alpha --i-table table.qza --p-metric simpson --o-alpha-diversity alpha-div_simpson.qza
```

## Evaluation signals

- An alpha-out TSV table exists for each processed dataset (datasets/dataset-name/alpha-out_dataset-name.tsv)
- Each alpha-out table contains the QIIME 2 alpha metric columns except ACE and Faith (when applied to synthetic datasets)
- On the 2x/10x/100x unevenness-ratio synthetic datasets, dominance-category metrics increase with the imposed ASV2/ASV1 ratio
- Conclusions drawn from non-rarefied metrics are reproduced when metrics are recomputed on rarefied datasets
- Metric values match those obtained by rerunning qiime diversity alpha on the same .qza table

## Limitations

- ACE and Faith could not be recalculated on synthetic datasets due to technical limitations (Faith is algorithmic and 'continuously suffering changes')
- Rarefied dataset availability is only described as 'data available in repository' without explicit accession or location
- Non-rarefied results should be interpreted knowing that all conclusions were also validated against rarefied datasets; skipping the rarefied cross-check leaves depth-related artifacts unexamined (e.g. the hypothesized subsampling artifact in the KarenThai cohort)

## Evidence

- [results] Alpha diversity metrics calculated with non-rarefied data; results also validated using rarefied datasets: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them. All results were also validated using rarefied datasets"
- [intro] Calculate alpha diversity metrics using QIIME2's diversity module: "All alpha diversity metrics were caluclated using QIIME2's diversity module."
- [readme] Aggregate alpha diversity metrics per dataset into a combined table: "All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
- [discussion] ACE and Faith metrics could not be recalculated on synthetic datasets due to technical limitations: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
