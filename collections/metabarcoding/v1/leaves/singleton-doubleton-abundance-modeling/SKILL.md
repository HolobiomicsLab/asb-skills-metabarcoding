---
name: singleton-doubleton-abundance-modeling
description: Use when validating how alpha diversity metrics respond to ASV count
  (50–500), singleton burden (0–100 or 40% of ASV count), and abundance-distribution
  shape, and when DADA2/DEBLUR singleton handling must be probed experimentally.
when_to_use_negative:
- Input is already a real ASV/feature table (e.g. from QIIME2 DADA2/DEBLUR output)
  — this skill generates synthetic data, not analysis of real samples
- The analysis requires recalculated ACE or Faith values on the synthetic data — the
  paper reports these could not be recalculated due to technical limitations
- The goal is rarefaction-based or phylogenetic analysis of real stool samples rather
  than controlled single/doubleton sensitivity testing
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0622
tools:
- name: QIIME2
  role: Platform in which the downstream alpha diversity metrics are recalculated
    on the generated synthetic datasets via its diversity module
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment manager hosting the QIIME2 setup and Python notebooks
    used for synthetic data generation
- name: DADA2
  role: Reference denoiser whose singleton-removal behavior motivates the explicit
    singleton modeling (contrast with DEBLUR)
- name: DEBLUR
  role: Reference denoiser that keeps singletons, defining the alternative singleton
    treatment the synthetic datasets emulate
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:39eda4ba5844524f2ff20c0e702d321d0a6f028dae8bc17c6634acf3c9c0901f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - QIIME2
  - anaconda
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# Singleton-doubleton abundance modeling

## Summary

Generates controlled synthetic sample-by-ASV count matrices (~2,500 samples per dataset, 7 datasets total) in which singleton ASVs are modeled explicitly and doubletons are assigned at 40% of the singleton count, mimicking the DADA2-vs-DEBLUR singleton treatment divergence, to validate alpha diversity metric behavior.

## When to use

Use when validating how alpha diversity metrics respond to ASV count (50–500), singleton burden (0–100 or 40% of ASV count), and abundance-distribution shape, and when DADA2/DEBLUR singleton handling must be probed experimentally.

## When NOT to use

- Input is already a real ASV/feature table (e.g. from QIIME2 DADA2/DEBLUR output) — this skill generates synthetic data, not analysis of real samples
- The analysis requires recalculated ACE or Faith values on the synthetic data — the paper reports these could not be recalculated due to technical limitations
- The goal is rarefaction-based or phylogenetic analysis of real stool samples rather than controlled single/doubleton sensitivity testing

## Inputs

- Parameter grid: ASV counts 50–500 (step 10), singleton counts 0–100 or 40% of ASV count (step 3)
- Abundance distribution choice: normal, negative binomial, uniform, exponential, or Poisson (NumPy/SciPy samplers)
- Artificial unevenness ratios (2x, 10x, 100x) for dominance-metric sensitivity testing
- Python environment with pinned dependencies from the alpha-diversities repository's requirements.txt

## Outputs

- 7 synthetic datasets, each a sample-by-ASV count matrix with ~2,500 samples, written as TSV abundance tables under the datasets directory
- Validated dataset structure (sample count, ASV count, singleton/doubleton composition) ready for alpha diversity metric recalculation

## How to apply

Define a parameter grid of 7 synthetic datasets with ~2,500 samples each, ASV counts from 50 to 500 in steps of 10, and singleton counts from 0 to 100 (or 40% of the ASV count) in steps of 3. For each configuration, model singleton ASVs explicitly and assign doubletons at 40% of the singleton count — a ratio chosen to reflect the known divergence where DADA2 removes singletons and DEBLUR keeps them. Draw the remaining ASV abundances from normal, negative binomial, uniform, exponential, or Poisson distributions using NumPy/SciPy samplers inside Jupyter notebooks (from the mauroibz/alpha-diversities environment with its pinned requirements.txt), running each cell sequentially. Also vary artificial unevenness ratios (2x, 10x, 100x) since dominance metrics are particularly sensitive to these. Write each resulting sample-by-ASV count matrix to a TSV abundance table under the datasets directory, then verify each dataset contains ~2,500 samples with the specified ASV, singleton, and doubleton structure before recalculating alpha diversity metrics on it.

## Related tools

- **QIIME2** (Platform in which the downstream alpha diversity metrics are recalculated on the generated synthetic datasets via its diversity module) — https://qiime2.org/
- **anaconda** (Virtual environment manager hosting the QIIME2 setup and Python notebooks used for synthetic data generation)
- **DADA2** (Reference denoiser whose singleton-removal behavior motivates the explicit singleton modeling (contrast with DEBLUR))
- **DEBLUR** (Reference denoiser that keeps singletons, defining the alternative singleton treatment the synthetic datasets emulate)

## Examples

```
python -m venv venv/; source venv/bin/activate; pip install -r requirements.txt; jupyter notebook --run-all-cells datasets/generate_synthetic.ipynb
```

## Evaluation signals

- Each generated dataset contains ~2,500 samples as specified in the parameter grid
- Dataset contains the specified ASV count (50–500 in steps of 10) and singleton count (0–100 or 40% of ASV count in steps of 3)
- Doubleton count equals 40% of the singleton count in each sample, per the design rule
- Metric ranges on synthetic datasets, after min-max normalization, show no meaningful differences across abundance distributions — divergence should localize to dominance metrics under 2x/10x/100x unevenness ratios
- ACE and Faith are correctly excluded from the downstream metric recalculation due to technical limitations

## Limitations

- ACE and Faith metrics could not be recalculated on the synthetic datasets due to technical limitations
- Dominance metrics are particularly affected by the artificial unevenness ratios (2x, 10x, 100x), so absolute (unnormalized) range comparisons across datasets are not meaningful
- The singleton/doubleton structure is a simplification reflecting DADA2 (removes singletons) vs DEBLUR (keeps them); interpretation depends on which denoise tool was used and should be reported as a source of variability in estimated richness

## Evidence

- [results] we validated our findings by applying the alpha diversity metrics to 7 synthetics datasets: "we validated our findings by applying the alpha diversity metrics to 7 synthetics datasets"
- [discussion] ACE and Faith exclusion on synthetic data: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
- [discussion] DADA2/DEBLUR singleton divergence motivating the design: "DADA2 removes singletons and DEBLUR keeps them."
- [readme] Notebook-based generation environment: "Exploration of data and vizualizations were carried out on different jupyter notebooks. Package requirements are on requirements.txt."
- [results] Min-max normalization finding on synthetic datasets: "after performing a min-max normalization to analyze their behavior, we did not see meaningful differences"
