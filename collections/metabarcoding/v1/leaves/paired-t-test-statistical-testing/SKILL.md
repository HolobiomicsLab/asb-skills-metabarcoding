---
name: paired-t-test-statistical-testing
description: Use when two classification conditions (e.g., species-level with bespoke
  weights vs. genus-level with uniform weights) have been evaluated on the same matched
  test samples and you must test whether per-sample error rates or Bray-Curtis dissimilarities
  differ significantly.
when_to_use_negative:
- The two error-rate vectors come from different, unmatched test samples — pairing
  is required for ttest_rel; use an unpaired test instead
- Comparing more than two classification conditions or weights (e.g., multiple cross-habitat
  weight sets) — a single paired t-test handles only one bespoke vs. uniform contrast
  at a time
- The input is already aggregated (e.g., a single mean error rate per condition) —
  the test requires the per-sample values
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0622
tools:
- name: scikit-learn
  role: underlying naive Bayes classifier machinery whose per-sample classification
    error rates form the paired vectors tested
- name: q2-feature-classifier
  role: classify-sklearn produces the predicted taxonomies under bespoke and uniform
    weight conditions from which per-sample error rates are computed
  repo: https://github.com/qiime2/q2-feature-classifier
- name: QIIME 2
  role: provides the Bray-Curtis dissimilarity computation between predicted and true
    community compositions used as a second paired response variable
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:1da6a6533b8085bea37457c668ec7c502486ffa1f05f11cbdc17c29f992a67d4
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# paired-t-test-statistical-testing

## Summary

A single-sided paired t-test that compares per-sample classification error rates (and Bray-Curtis dissimilarities) from bespoke species-level taxonomic weighting against uniform genus-level classification in q2-feature-classifier, testing whether bespoke weights significantly improve species-level accuracy.

## When to use

Use when two classification conditions (e.g., species-level with bespoke weights vs. genus-level with uniform weights) have been evaluated on the same matched test samples and you must test whether per-sample error rates or Bray-Curtis dissimilarities differ significantly.

## When NOT to use

- The two error-rate vectors come from different, unmatched test samples — pairing is required for ttest_rel; use an unpaired test instead
- Comparing more than two classification conditions or weights (e.g., multiple cross-habitat weight sets) — a single paired t-test handles only one bespoke vs. uniform contrast at a time
- The input is already aggregated (e.g., a single mean error rate per condition) — the test requires the per-sample values

## Inputs

- per-sample species-level classification error rates from bespoke-weight classify-sklearn runs on Greengenes 13_8 99% OTU reference
- per-sample genus-level classification error rates from uniform-weight classify-sklearn runs on the same matched test samples
- per-sample Bray-Curtis dissimilarity values between predicted and true community compositions for each condition

## Outputs

- t-statistic and single-sided P-value from the paired t-test on error rates
- t-statistic and single-sided P-value from the paired t-test on Bray-Curtis dissimilarities
- fig1_comparison.csv with mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values

## How to apply

After classifying the same test amplicon sequences under both conditions with q2-feature-classifier's classify-sklearn against the Greengenes 13_8 99% OTU reference, compute per-sample error rates by comparing predicted taxonomies to known ground-truth taxonomies (species-level for the bespoke condition, genus-level for the uniform condition). Keep the per-sample pairing intact: each test sample contributes one error rate and one Bray-Curtis dissimilarity under each condition, and the pairing must be preserved between the two vectors. Use a one-sided (single-sided) paired t-test — e.g., scipy.stats.ttest_rel with alternative='greater' — testing whether species-level bespoke error rates are lower than genus-level uniform error rates across the 14 EMPO 3 habitat types; run the same test on the matched Bray-Curtis dissimilarity values. Compile the mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values (expected: 14% vs 16% error, 0.13 vs 0.15 dissimilarity, P = 0.013, matching the paper's Fig. 1) into fig1_comparison.csv.

## Related tools

- **scikit-learn** (underlying naive Bayes classifier machinery whose per-sample classification error rates form the paired vectors tested)
- **q2-feature-classifier** (classify-sklearn produces the predicted taxonomies under bespoke and uniform weight conditions from which per-sample error rates are computed) — https://github.com/qiime2/q2-feature-classifier
- **QIIME 2** (provides the Bray-Curtis dissimilarity computation between predicted and true community compositions used as a second paired response variable)

## Examples

```
from scipy.stats import ttest_rel; stat, p = ttest_rel(bespoke_species_err, uniform_genus_err, alternative='greater')
```

## Evaluation signals

- Mean species-level bespoke error rate is approximately 14% and genus-level uniform error rate approximately 16% across the 14 EMPO 3 habitat types
- Mean Bray-Curtis dissimilarity is 0.13 (bespoke species) vs 0.15 (uniform genus)
- Single-sided paired t-test P = 0.013 for error rates, matching Fig. 1 of the paper
- The two response vectors have equal length and are ordered by the same matched test-sample IDs
- fig1_comparison.csv contains mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values for both conditions

## Limitations

- The paper notes some degradation in performance for rare species under bespoke weighting, which could weaken the paired improvement signal
- Empirical species distributions introduce potential error sources such as misannotated sample types and sequencing biases propagated into taxonomic weights, adding noise to the per-sample rates being tested
- The test is one-sided and pairwise over matched samples; the 14-sample-scale replication (per EMPO 3 habitat type) limits power if habitat variance is large

## Evidence

- [methods] workflow t-test step: [evidence span withheld — anchor status: not_found]
- [methods] validation targets: [evidence span withheld — anchor status: not_found]
- [discussion] directional claim the test supports: "demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy"
- [readme] classifier and weights provenance: [evidence span withheld — anchor status: external_doc]
