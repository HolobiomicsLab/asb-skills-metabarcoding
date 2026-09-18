---
name: class-weighted-species-level-error-rate-comparison
description: Use when you need to quantify how divergent taxonomic class weights (e.g.,
  cross-habitat EMPO 3 weights or uniform weights) inflate species-level classification
  error relative to source-habitat weights in QIIME 2.
when_to_use_negative:
- When the sample's true source-habitat bespoke weights are already available and
  you simply want the best classification — use those directly rather than benchmarking
  cross-habitat degradation
- For samples from environments lacking sufficient characterization for bespoke weight
  assembly — the article recommends average weights from global microbial species
  distributions instead of uniform weights
- For highly unusual sample distributions (e.g., synthetic populations) — compile
  custom bespoke weights from existing samples rather than reusing habitat-level weights
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: q2-clawback
  role: Assembles the taxonomic class weights (from each of the other 13 habitats
    and controls) applied to the classifier
  repo: https://github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Framework hosting the q2-feature-classifier and q2-clawback plugins used to
    train and run the weighted classifier
- name: q2-feature-classifier
  role: Trains the naive Bayes classifier with class weights and classifies habitat
    samples
- name: scikit-learn
  role: Provides the multinomial naive Bayes classifier and the linear regression/Pearson
    correlation used to model error vs divergence
- name: imbalanced-learn
  role: Conventions for enabling class weights in the naive Bayes classifier
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:fba23c3c1b55fa37a2bbf6130950737604cf49f5661ac5d3e618a8c6f59ad6d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# class-weighted-species-level-error-rate-comparison

## Summary

Benchmark how species-level naive Bayes classification error rates change when taxonomic class weights from a mismatched (cross-habitat) source are applied instead of bespoke habitat weights, and quantify degradation via linear regression of error rate on weight divergence.

## When to use

Use when you need to quantify how divergent taxonomic class weights (e.g., cross-habitat EMPO 3 weights or uniform weights) inflate species-level classification error relative to source-habitat weights in QIIME 2.

## When NOT to use

- When the sample's true source-habitat bespoke weights are already available and you simply want the best classification — use those directly rather than benchmarking cross-habitat degradation
- For samples from environments lacking sufficient characterization for bespoke weight assembly — the article recommends average weights from global microbial species distributions instead of uniform weights
- For highly unusual sample distributions (e.g., synthetic populations) — compile custom bespoke weights from existing samples rather than reusing habitat-level weights

## Inputs

- EMPO 3 habitat sample sequence variants (per-habitat feature tables)
- Greengenes reference sequences and taxonomy
- q2-clawback class weights per habitat (TaxonomicWeights artifacts) assembled from other habitats
- Uniform-weights control configuration

## Outputs

- Species-level error rate matrix for all habitat x weight-set combinations (182 comparisons)
- Win tally of cross-habitat vs uniform comparisons (expected 117/182)
- Linear regression and Pearson correlation statistics (r2 = 0.57, P < 2.2e-16)
- figures_3_4_results.csv and Fig. 3-4 equivalent PNG plots

## How to apply

Train a scikit-learn multinomial naive Bayes classifier with q2-feature-classifier on Greengenes reference sequences/taxonomy with class weights enabled (imbalanced-learn conventions). For each of the 14 EMPO 3 habitats, classify its samples using q2-clawback class weights assembled from each of the other 13 habitats, plus a uniform-weights control, yielding 182 cross-habitat-versus-uniform comparisons. Compute species-level error rates per habitat x weight-set combination, tally wins (expected: cross-habitat weights beat uniform in 117 of 182 comparisons), then fit a linear regression of error rate on weight divergence and verify Pearson r2 = 0.57 (P < 2.2e-16). The rationale is that classification accuracy degrades linearly with the degree to which the 'all reference species equally likely' assumption is violated, so even deliberately incorrect weights should outperform uniform weights.

## Related tools

- **q2-clawback** (Assembles the taxonomic class weights (from each of the other 13 habitats and controls) applied to the classifier) — https://github.com/BenKaehler/q2-clawback
- **QIIME 2** (Framework hosting the q2-feature-classifier and q2-clawback plugins used to train and run the weighted classifier)
- **q2-feature-classifier** (Trains the naive Bayes classifier with class weights and classifies habitat samples)
- **scikit-learn** (Provides the multinomial naive Bayes classifier and the linear regression/Pearson correlation used to model error vs divergence)
- **imbalanced-learn** (Conventions for enabling class weights in the naive Bayes classifier)

## Examples

```
qiime clawback classify-sklearn-weights --i-classifier gg-nb-classifier.qza --i-weights cross-habitat-weights.qza --i-reads empo3-habitat-reads.qza --o-classification habitat-predictions.qza
```

## Evaluation signals

- Cross-habitat weights outperform uniform weights in the majority of comparisons — verify the 117 of 182 tally
- Linear regression of species-level error rate on weight divergence recovers Pearson r2 = 0.57 with P < 2.2e-16
- Species-level error rates fall within the reported range (average error rates decline from 25% to 14% under bespoke weighting)
- 182 total comparisons (14 habitats x 13 cross-habitat weight sets) present in the comparison table
- figures_3_4_results.csv contains the comparison table, win tally, and regression statistics

## Limitations

- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation)
- Empirical species distributions introduce potential error sources such as misannotated sample types and sequencing biases propagated into taxonomic weights
- Some degradation in performance was observed for rare species under bespoke weighting

## Evidence

- [results] cross-habitat error rate trend: "We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated."
- [discussion] uniform vs incorrect weights: "We have demonstrated that this assumption imposes a consistently negative impact on performance, even when compared to deliberately incorrect taxonomic weights"
- [methods] workflow tools: [evidence span withheld — anchor status: not_found]
- [readme] plugin purpose: [evidence span withheld — anchor status: not_found]
- [readme] readytowear habitats: [evidence span withheld — anchor status: external_doc]
