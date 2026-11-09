---
name: asv-count-parameterization
description: Use when validating alpha diversity metrics (richness, dominance, entropy,
  etc.) on data with controlled ASV counts, singleton/doubleton ratios, or artificial
  unevenness ratios (2x, 10x, 100x), rather than only on real microbiome samples.
when_to_use_negative:
- Input is already a real (non-synthetic) amplicon feature table from DADA2/DEBLUR
  processing — use QIIME2's diversity module instead
- You need to recalculate ACE or Faith metrics on the synthetic data — these were
  excluded due to technical limitations
- The analysis goal is empirical correlation among metrics on real stool samples (4,596
  samples from 13 projects), not controlled composition validation
edam_topics:
- http://edamontology.org/topic_3307
tools:
- name: QIIME2
  role: Upstream pipeline whose denoising tools (DADA2/DEBLUR) motivate the singleton/doubleton
    composition being simulated; QIIME2's diversity module recalculates metrics on
    the synthetic tables
  repo: https://qiime2.org/
- name: anaconda
  role: Virtual environment manager for the Python/QIIME2 environment in which the
    synthetic datasets are generated and processed
  repo: https://qiime2.org/
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:26d751f7cf0badd1e430354527b352f0e2a355d3a35c232682d1142d595b4846
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME2
  - anaconda
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# ASV count parameterization

## Summary

Generate synthetic sample-by-ASV abundance tables with controlled ASV, singleton, and doubleton composition so alpha diversity metric behavior can be validated without relying solely on real stool data. It reflects the DADA2 (removes singletons) vs DEBLUR (keeps them) divergence.

## When to use

Use when validating alpha diversity metrics (richness, dominance, entropy, etc.) on data with controlled ASV counts, singleton/doubleton ratios, or artificial unevenness ratios (2x, 10x, 100x), rather than only on real microbiome samples.

## When NOT to use

- Input is already a real (non-synthetic) amplicon feature table from DADA2/DEBLUR processing — use QIIME2's diversity module instead
- You need to recalculate ACE or Faith metrics on the synthetic data — these were excluded due to technical limitations
- The analysis goal is empirical correlation among metrics on real stool samples (4,596 samples from 13 projects), not controlled composition validation

## Inputs

- Parameter grid specification (ASV counts 50-500 step 10; singleton counts 0-100 or 40% of ASV count, step 3; doubletons at 40% of singleton count)
- Choice of abundance distribution (normal, negative binomial, uniform, exponential, or Poisson)
- Python virtual environment with pinned dependencies from requirements.txt

## Outputs

- Seven synthetic sample-by-ASV count matrices (~2,500 samples each) written as TSV abundance tables under the datasets directory
- Validation check confirming ASV, singleton, and doubleton structure per dataset

## How to apply

Define a parameter grid over ~2,500 samples per dataset, with ASV counts from 50 to 500 in steps of 10 and singleton counts from 0 to 100 (or 40% of the ASV count) in steps of 3. Explicitly model singleton ASVs and assign doubletons at 40% of the singleton count, mirroring the known DADA2 vs DEBLUR singleton-treatment divergence. Draw remaining ASV abundances from normal, negative binomial, uniform, exponential, or Poisson distributions using NumPy/SciPy samplers inside Jupyter notebooks, run sequentially in a pinned Python venv. Write each synthetic sample-by-ASV count matrix to a TSV abundance table under the datasets directory of the alpha-diversities repository. Validate that each generated dataset contains ~2,500 samples with the specified ASV, singleton, and doubleton structure before recalculating alpha diversity metrics (ACE and Faith excluded due to technical limitations).

## Related tools

- **QIIME2** (Upstream pipeline whose denoising tools (DADA2/DEBLUR) motivate the singleton/doubleton composition being simulated; QIIME2's diversity module recalculates metrics on the synthetic tables) — https://qiime2.org/
- **anaconda** (Virtual environment manager for the Python/QIIME2 environment in which the synthetic datasets are generated and processed) — https://qiime2.org/

## Examples

```
source venv/bin/activate && pip install -r requirements.txt
```

## Evaluation signals

- Each generated dataset contains ~2,500 samples
- ASV counts per sample span 50-500 in steps of 10 as defined by the grid
- Doubleton count equals 40% of the singleton count in every configuration
- All seven datasets written as TSVs under datasets/ before metric recalculation, with ACE and Faith excluded
- After min-max normalization, non-dominance metrics show no meaningful differences across abundance distributions, while dominance metrics respond to the 2x, 10x, and 100x unevenness ratios

## Limitations

- ACE and Faith metrics could not be recalculated on the synthetic datasets due to technical limitations
- Synthetic unevenness ratios (2x, 10x, 100x) are artificial; dominance metrics were particularly sensitive to them, so results may not transfer directly to natural abundance distributions
- Singleton counts capped at 0-100 (or 40% of ASV count) in steps of 3, so extreme singleton fractions outside this grid are untested

## Evidence

- [results] Parameter grid for the 7 synthetic datasets: [evidence span withheld — anchor status: not_found]
- [discussion] Singleton/doubleton rationale: "DADA2 removes singletons and DEBLUR keeps them."
- [results] Validation scope: "we validated our findings by applying the alpha diversity metrics to 7 synthetics datasets"
- [readme] Environment setup: "To run the notebooks you can perform the following steps:
    Build a python virtual enviroment"
- [discussion] ACE and Faith exclusion: "we recalculated the alpha diversity metrics for all, with the exception of ACE and Faith due to technical limitations."
