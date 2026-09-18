---
name: parametric-count-distribution-sampling
description: Use when building controlled synthetic microbiome abundance tables to
  benchmark or validate alpha diversity metrics, especially when singleton/doubleton
  treatment (DADA2 vs DEBLUR divergence) must be explicitly controlled.
when_to_use_negative:
- Input is already a real amplicon feature table (e.g. QIIME2 BIOM/TSV from DADA2
  or DEBLUR) — this skill only creates synthetic tables for metric validation, not
  real-data processing
- The analysis requires ACE or Faith on the synthetic data — these could not be recalculated
  on synthetic datasets due to technical limitations
- The goal is to compare denoiser outputs (DADA2 vs DEBLUR) on real sequencing runs
  rather than to probe metric sensitivity to controlled singleton/doubleton structure
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: QIIME2
  role: Suite on whose alpha diversity metrics (diversity module) the synthetic count
    matrices are re-evaluated for validation; environment via anaconda
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment manager in which QIIME2 and the analysis notebooks are
    run
  repo: https://qiime2.org/
- name: sra-tools
  role: Used in the same workflow to fetch real SRA datasets that complement the synthetic
    validation datasets
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:b755fc0e5367cf0093376072c00e64902bfc50256dde0af4b83436dd2bd5ca47
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# parametric-count-distribution-sampling

## Summary

Generate synthetic sample-by-ASV count matrices by sampling ASV abundances from parametric distributions (normal, negative binomial, uniform, exponential, Poisson) with explicitly modeled singleton and doubleton counts, to validate the behavior of alpha diversity metrics under controlled abundance structures.

## When to use

Use when building controlled synthetic microbiome abundance tables to benchmark or validate alpha diversity metrics, especially when singleton/doubleton treatment (DADA2 vs DEBLUR divergence) must be explicitly controlled.

## When NOT to use

- Input is already a real amplicon feature table (e.g. QIIME2 BIOM/TSV from DADA2 or DEBLUR) — this skill only creates synthetic tables for metric validation, not real-data processing
- The analysis requires ACE or Faith on the synthetic data — these could not be recalculated on synthetic datasets due to technical limitations
- The goal is to compare denoiser outputs (DADA2 vs DEBLUR) on real sequencing runs rather than to probe metric sensitivity to controlled singleton/doubleton structure

## Inputs

- Parameter grid: ASV counts (50–500, step 10), singleton counts (0–100 or 40% of ASV count, step 3), target ~2,500 samples per dataset
- Chosen parametric abundance distribution (normal, negative binomial, uniform, exponential, or Poisson) with NumPy/SciPy sampler parameters
- requirements.txt from the mauroibz/alpha-diversities repository (pinned Python environment)

## Outputs

- Seven synthetic sample-by-ASV count matrices written as TSV abundance tables under the datasets directory
- Validated dataset structure: ~2,500 samples with specified ASV, singleton, and doubleton composition per dataset

## How to apply

Define a parameter grid over ASV counts (50 to 500 in steps of 10) and singleton counts (0 to 100, or 40% of the ASV count, in steps of 3), producing ~2,500 samples per synthetic dataset for 7 datasets. For each configuration, model singleton ASVs explicitly and assign doubletons at 40% of the singleton count, reflecting the DADA2-removes/DEBLUR-keeps singleton divergence. Draw the remaining ASV abundances from normal, negative binomial, uniform, exponential, or Poisson distributions using NumPy/SciPy samplers executed sequentially inside Jupyter notebooks from the alpha-diversities repository (python venv + pip install -r requirements.txt). Write each sample-by-ASV count matrix as a TSV abundance table under the datasets directory. Optionally vary artificial unevenness ratios (2x, 10x, 100x) to stress dominance metrics.

## Related tools

- **QIIME2** (Suite on whose alpha diversity metrics (diversity module) the synthetic count matrices are re-evaluated for validation; environment via anaconda) — https://qiime2.org/
- **anaconda** (Virtual environment manager in which QIIME2 and the analysis notebooks are run) — https://qiime2.org/
- **sra-tools** (Used in the same workflow to fetch real SRA datasets that complement the synthetic validation datasets) — https://github.com/ncbi/sra-tools

## Examples

```
python -m venv venv/; source venv/bin/activate; pip install -r requirements.txt; jupyter notebook  # run synthetic-data notebooks sequentially to emit TSV count matrices
```

## Evaluation signals

- Each generated dataset contains ~2,500 samples
- ASV counts per sample span the grid 50–500 in steps of 10
- Singleton counts match the grid (0–100, or 40% of ASV count, in steps of 3) and doubletons equal ~40% of the singleton count
- Count matrices are valid TSV abundance tables loadable downstream for alpha diversity metric recalculation (all metrics except ACE and Faith)
- Min-max normalized metric values across distributions show no meaningful differences, while dominance metrics respond to the 2x/10x/100x unevenness ratios — confirming the controlled abundance structure took effect

## Limitations

- ACE and Faith metrics could not be recalculated on the synthetic datasets due to technical limitations
- Findings from synthetic data depend on the artificial unevenness ratios (2x, 10x, 100x), which particularly affect dominance metrics
- Notebook execution requires running each cell in sequential order in a Python virtual environment built from requirements.txt; results validated on non-rarefied data

## Evidence

- [methods] Draw the remaining ASV abundances from normal, negative binomial, uniform, exponential, or Poisson distributions using NumPy/SciPy samplers within the Jupyter notebooks: [evidence span withheld — anchor status: not_found]
- [methods] Singleton/doubleton design rationale: [evidence span withheld — anchor status: not_found]
- [results] Dominance metrics sensitivity: [evidence span withheld — anchor status: not_found]
- [discussion] ACE and Faith exclusion: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
- [readme] Notebook environment setup: "Build a python virtual enviroment
    > python -m venv venv/

    Activate the python virtual enviroment
    > source venv/bin/activate

    Install requirments
    > pip install -r requirements.txt"
