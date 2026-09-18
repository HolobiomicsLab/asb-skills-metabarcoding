---
name: synthetic-microbiome-abundance-generation
description: Use when validating alpha diversity metric behavior (richness, dominance,
  entropy, phylogenetic) on data with fully controlled ASV counts, singleton/doubleton
  composition, and abundance distributions rather than on real microbiome samples.
when_to_use_negative:
- The goal is to compare metrics on real empirical data — use the 13 publicly available
  human microbiome projects (4,596 stool samples) downloaded from SRA instead.
- ACE or Faith must be recalculated on the synthetic data — these are excluded due
  to technical limitations (Faith is algorithmic and 'continuously suffering changes').
- The input is already a denoised QIIME2 feature table from DADA2 or DEBLUR — this
  skill generates count data from scratch, not from processed amplicon outputs.
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Suite whose diversity module provides the 19 alpha diversity metrics that
    the synthetic datasets are used to validate; workflows ran on QIIME 2022.2.0.
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment platform used for all sample processing and analysis in
    the alpha-diversities workflow.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:3e484d30bba9e3b48279d59abc1fc56ee278527e2455942a4ad35671a1f117c3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# synthetic-microbiome-abundance-generation

## Summary

Generate seven synthetic sample-by-ASV count matrices (~2,500 samples each) with controlled ASV, singleton, and doubleton composition, used to validate the behavior of alpha diversity metrics under known abundance distributions.

## When to use

Use when validating alpha diversity metric behavior (richness, dominance, entropy, phylogenetic) on data with fully controlled ASV counts, singleton/doubleton composition, and abundance distributions rather than on real microbiome samples.

## When NOT to use

- The goal is to compare metrics on real empirical data — use the 13 publicly available human microbiome projects (4,596 stool samples) downloaded from SRA instead.
- ACE or Faith must be recalculated on the synthetic data — these are excluded due to technical limitations (Faith is algorithmic and 'continuously suffering changes').
- The input is already a denoised QIIME2 feature table from DADA2 or DEBLUR — this skill generates count data from scratch, not from processed amplicon outputs.

## Inputs

- Parameter grid specification: ~2,500 samples/dataset, ASV counts 50–500 (step 10), singleton counts 0–100 or 40% of ASV count (step 3)
- NumPy/SciPy distribution samplers (normal, negative binomial, uniform, exponential, Poisson)
- Pinned dependencies from requirements.txt of the alpha-diversities repository
- Jupyter notebooks with cells run in sequential order

## Outputs

- 7 synthetic sample-by-ASV count matrices written as TSV abundance tables under the datasets directory
- Validated synthetic datasets with controlled singleton/doubleton structure for downstream alpha diversity metric recalculation

## How to apply

Set up the environment from the mauroibz/alpha-diversities repository by building a Python virtual environment and installing the pinned requirements.txt, then run the Jupyter notebooks cells sequentially. Define a parameter grid of 7 synthetic datasets with ~2,500 samples per dataset, ASV counts from 50 to 500 in steps of 10, and singleton counts from 0 to 100 (or 40% of the ASV count) in steps of 3. Model singletons explicitly and assign doubletons at 40% of the singleton count, a choice grounded in the DADA2 (removes singletons) vs DEBLUR (keeps them) divergence. Draw remaining ASV abundances from normal, negative binomial, uniform, exponential, or Poisson distributions using NumPy/SciPy samplers, including artificial unevenness ratios of 2x, 10x, and 100x to probe dominance metrics. Write each synthetic sample-by-ASV count matrix as a TSV abundance table under the datasets directory, then validate sample counts and singleton/doubleton structure before recalculating the alpha diversity metrics (all except ACE and Faith, which are excluded due to technical limitations).

## Related tools

- **QIIME2** (Suite whose diversity module provides the 19 alpha diversity metrics that the synthetic datasets are used to validate; workflows ran on QIIME 2022.2.0.) — https://qiime2.org/
- **anaconda** (Virtual environment platform used for all sample processing and analysis in the alpha-diversities workflow.)

## Examples

```
python -m venv venv/; source venv/bin/activate; pip install -r requirements.txt
```

## Evaluation signals

- Each generated dataset contains ~2,500 samples.
- ASV counts follow the grid from 50 to 500 in steps of 10.
- Singleton counts span 0 to 100 (or 40% of the ASV count) in steps of 3, with doubletons fixed at 40% of the singleton count.
- Each synthetic dataset is written as a TSV abundance table under the datasets directory.
- Downstream metric recalculation succeeds for all metrics except ACE and Faith, and dominance metrics show sensitivity to the 2x, 10x, and 100x unevenness ratios.

## Limitations

- ACE and Faith could not be recalculated on the synthetic datasets due to technical limitations.
- Dominance metrics were particularly affected by the artificial unevenness ratios, so conclusions about them on synthetic data reflect the imposed 2x, 10x, and 100x structure rather than natural distributions.
- On the synthetic datasets, most metrics' value ranges changed with the abundance distribution but showed no meaningful differences after min-max normalization, limiting discrimination between metrics in this setting.

## Evidence

- [readme] All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment.: "All sample processing was done using QIIME2 https://qiime2.org/ on an anaconda virtual enviroment."
- [results] Synthetic datasets as validation: "we validated our findings by applying the alpha diversity metrics to 7 synthetics datasets"
- [discussion] ACE and Faith exclusion: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
- [discussion] Singleton treatment divergence motivating doubleton design: "DADA2 removes singletons and DEBLUR keeps them."
- [readme] Notebook environment setup from README: "Install requirments
    > pip install -r requirements.txt"
