---
name: paired-hypothesis-testing-of-classifier-performance
description: Use when two classifiers or weighting schemes are evaluated on the same
  paired units (e.g., the same 14 EMPO 3 habitats) and you need to test whether the
  per-unit error-rate difference is statistically significant.
when_to_use_negative:
- The two performance sets are not paired (e.g., classifiers evaluated on different
  query samples or different habitats) — an unpaired or nonparametric test is required
  instead
- Only a single classification run exists with no competing weighting scheme or baseline
  to compare against
- The comparison is of abundance/distribution estimates rather than classification
  error rates measured on the same units
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: q2-feature-classifier
  role: Trains the multinomial naive Bayes classifiers (fit-classifier-naive-bayes)
    and classifies held-out ASVs (classify-sklearn, confidence 0.7) to produce the
    paired error rates being tested
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: scikit-learn
  role: Provides MultinomialNB with class_weight='balanced' used to implement bespoke
    vs. uniform class weighting in the classifiers under comparison
- name: q2-clawback
  role: Assembles the bespoke habitat-specific taxonomic class weights that constitute
    the treatment arm of the paired comparison
  repo: github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Framework in which the classifier plugin, weights, and classification runs
    are orchestrated
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:83ee5f7363dab989ac67151bba7fef5718826ae88b3d4602832d41d141faab3f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qiime2/q2-feature-classifier
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# Paired hypothesis testing of classifier performance

## Summary

Statistical test that compares two taxonomic classification schemes (bespoke EMPO 3 habitat-specific weights vs. uniform weights) evaluated on the same paired units, using a paired t-test over per-habitat species-level error rates to establish whether environment-specific weights significantly improve accuracy.

## When to use

Use when two classifiers or weighting schemes are evaluated on the same paired units (e.g., the same 14 EMPO 3 habitats) and you need to test whether the per-unit error-rate difference is statistically significant.

## When NOT to use

- The two performance sets are not paired (e.g., classifiers evaluated on different query samples or different habitats) — an unpaired or nonparametric test is required instead
- Only a single classification run exists with no competing weighting scheme or baseline to compare against
- The comparison is of abundance/distribution estimates rather than classification error rates measured on the same units

## Inputs

- Per-habitat species-level error rates for the bespoke (EMPO 3 habitat-specific class_weight='balanced') classifier
- Per-habitat species-level error rates for the uniform-weight baseline classifier over the identical held-out query ASV sets
- Greengenes 99% OTU reference sequences and taxonomy trained classifiers (one per weighting scheme)
- fig1_error_rates.csv (per-habitat and overall average error-rate table)

## Outputs

- Paired t-test statistic and P-value comparing bespoke vs. uniform per-habitat error rates (reported P = 5.8e-5)
- Tabulated per-habitat and overall average error rates (14% bespoke vs. 25% uniform)
- fig1_error_rates.csv results file

## How to apply

First generate paired measurements: train the q2-feature-classifier fit-classifier-naive-bayes multinomial naive Bayes classifier (scikit-learn MultinomialNB, class_weight='balanced') once per habitat with bespoke class weights and once with uniform weights, then classify each habitat's held-out query amplicon sequence variants with classify-sklearn at confidence 0.7 under both schemes. Compute species-level classification error rates for every EMPO 3 habitat for both weighting schemes, keeping the habitat labels as the pairing key. Run a paired t-test (e.g., scipy.stats.ttest_rel) on the 14 bespoke-vs-uniform error-rate pairs, since the two schemes share identical query/reference units per habitat. Tabulate per-habitat and overall average error rates and save to fig1_error_rates.csv; the reproduction target is 14% (bespoke) vs 25% (uniform) with paired t-test P = 5.8e-5. Report the effect in context: bespoke species-level error (14%) is favorably comparable to existing classifiers' genus-level error rate of 16%.

## Related tools

- **q2-feature-classifier** (Trains the multinomial naive Bayes classifiers (fit-classifier-naive-bayes) and classifies held-out ASVs (classify-sklearn, confidence 0.7) to produce the paired error rates being tested) — https://github.com/qiime2/q2-feature-classifier/
- **scikit-learn** (Provides MultinomialNB with class_weight='balanced' used to implement bespoke vs. uniform class weighting in the classifiers under comparison)
- **q2-clawback** (Assembles the bespoke habitat-specific taxonomic class weights that constitute the treatment arm of the paired comparison) — github.com/BenKaehler/q2-clawback
- **QIIME 2** (Framework in which the classifier plugin, weights, and classification runs are orchestrated)

## Examples

```
python -c "import pandas as pd; from scipy.stats import ttest_rel; df = pd.read_csv('fig1_error_rates.csv'); print(ttest_rel(df['bespoke_error'], df['uniform_error']))"
```

## Evaluation signals

- Direction check: bespoke per-habitat species-level error rate is lower than uniform for the paired comparison to support the hypothesis
- Reproduced overall averages fall near 14% (bespoke) vs. 25% (uniform) across the 14 EMPO 3 habitats
- Paired t-test P-value reproduces the reported 5.8 × 10−5 (same order of magnitude on the same 14 pairs)
- Pairing integrity: exactly 14 habitat pairs, same habitat labels and same held-out ASV sets under both weighting schemes
- Contextual sanity: bespoke species-level error (~14%) should be favorably comparable to the 16% genus-level error rate of existing classifiers

## Limitations

- Some degradation in performance for rare species was observed under bespoke weighting, so aggregate error-rate improvement may mask rare-species regressions
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation) or misannotated sample types and sequencing biases propagated into the taxonomic weights
- The comparison relies on per-habitat pairing across 14 EMPO 3 habitats; conclusions apply to common sample types with characterized environments

## Evidence

- [abstract] Species-level average error rates decline from 25% to 14%: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [discussion] Uniform assumption consistently negative: "We have demonstrated that this assumption imposes a consistently negative impact on performance, even when compared to deliberately incorrect taxonomic weights"
- [results] Paired t-test result: [evidence span withheld — anchor status: not_found]
- [readme] Ready-made weights source: "please check out our ready-made weights at [readytowear](https://github.com/BenKaehler/readytowear)"
