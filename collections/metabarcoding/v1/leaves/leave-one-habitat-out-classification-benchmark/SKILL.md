---
name: leave-one-habitat-out-classification-benchmark
description: Use when you need to measure how much bespoke (source-environment) taxonomic
  weights matter, by benchmarking mis-matched or 'deliberately incorrect' habitat
  weights against uniform weights across habitat classes.
when_to_use_negative:
- Your query samples already have a bespoke weight set for their true source habitat
  — apply the correct bespoke weights directly rather than benchmarking mis-matched
  weights
- Your samples come from habitats not represented in the weight inventory (e.g. highly
  unusual distributions like synthetic populations), where no cross-habitat weight
  assembly is possible
- You only need genus-level classification, where the equal-abundance assumption's
  impact is much smaller than at species level
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: q2-clawback
  role: Assembles taxonomic class weights per habitat that are used to re-weight the
    naive Bayes classifier
  repo: github.com/BenKaehler/q2-clawback
- name: q2-feature-classifier
  role: Trains and runs the multinomial naive Bayes classifier used to classify sequences
    under each weight set
- name: scikit-learn
  role: Provides the multinomial naive Bayes classifier and the linear regression
    fit of error rate on weight divergence
- name: imbalanced-learn
  role: Conventions for enabling class weights in the naive Bayes classifier
- name: QIIME 2
  role: Plugin framework within which classifier training, weighting, and classification
    are executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:6685021a110d1c238ace5afa54e5ffc9eb193696917ed26212d15a253fbd6444
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# leave-one-habitat-out-classification-benchmark

## Summary

A benchmark that quantifies how species-level naive Bayes classification degrades when class weights from the wrong habitat are applied: for each of 14 EMPO 3 habitats, samples are classified with q2-clawback weights from each of the other 13 habitats plus a uniform-weights control, and error rates are regressed against weight divergence.

## When to use

Use when you need to measure how much bespoke (source-environment) taxonomic weights matter, by benchmarking mis-matched or 'deliberately incorrect' habitat weights against uniform weights across habitat classes.

## When NOT to use

- Your query samples already have a bespoke weight set for their true source habitat — apply the correct bespoke weights directly rather than benchmarking mis-matched weights
- Your samples come from habitats not represented in the weight inventory (e.g. highly unusual distributions like synthetic populations), where no cross-habitat weight assembly is possible
- You only need genus-level classification, where the equal-abundance assumption's impact is much smaller than at species level

## Inputs

- EMPO 3 habitat sample sequence variants (feature tables / sequence variants per habitat)
- Greengenes reference sequences and taxonomy
- q2-clawback class weights per habitat (assembled from the other 13 habitats) and a uniform-weights set

## Outputs

- Species-level classification error rates for every habitat x weight-set combination (182 cross-habitat-versus-uniform comparisons)
- Comparison table, win tally, and regression statistics (figures_3_4_results.csv)
- Fig. 3-4 equivalent PNG plots (error-rate vs weight-divergence linear trend)

## How to apply

Load the EMPO 3 habitat sample sequence variants and Greengenes reference sequences/taxonomy, then train a naive Bayes classifier with QIIME 2's q2-feature-classifier (scikit-learn multinomial naive Bayes with class weights enabled via imbalanced-learn conventions). For each of the 14 EMPO 3 habitats in turn, classify its samples using q2-clawback class weights assembled from each of the other 13 habitats, plus a uniform-weights control run, yielding 182 habitat x weight-set combinations. Compute species-level classification error rates for every combination, then tally the comparisons where cross-habitat weights beat uniform weights and fit a linear regression of species-level error rate on weight divergence with scikit-learn. Rationale: the paper shows classification accuracy degrades linearly with the degree to which the equal-abundance assumption is violated, so this design tests both the majority win rate of incorrect-but-informed weights and the linear error trend.

## Related tools

- **q2-clawback** (Assembles taxonomic class weights per habitat that are used to re-weight the naive Bayes classifier) — github.com/BenKaehler/q2-clawback
- **q2-feature-classifier** (Trains and runs the multinomial naive Bayes classifier used to classify sequences under each weight set)
- **scikit-learn** (Provides the multinomial naive Bayes classifier and the linear regression fit of error rate on weight divergence)
- **imbalanced-learn** (Conventions for enabling class weights in the naive Bayes classifier)
- **QIIME 2** (Plugin framework within which classifier training, weighting, and classification are executed)

## Examples

```
redbiom search taxon --context $ctx s__aureus | head | redbiom search features --context $ctx
```

## Evaluation signals

- Cross-habitat weights beat uniform weights in 117 of 182 comparisons — the tally should reproduce this majority
- Linear regression of species-level error rate on weight divergence recovers Pearson r2 = 0.57 with P < 2.2e-16
- The error-rate vs weight-divergence relationship is linear (degrades linearly as applied weights diverge from source-habitat weights)
- 182 total comparisons generated (14 habitats x 13 other-habitat weight sets)
- figures_3_4_results.csv contains the comparison table, win tally, and regression statistics as expected

## Limitations

- Cross-habitat weights are deliberately incorrect weights; results demonstrate that even these outperform uniform weights, but the most specific weights to a query sample's environment always give best accuracy
- Some degradation in performance was observed for rare species under bespoke weighting
- Empirical species distributions introduce error sources such as misannotated sample types or sequencing biases propagated into taxonomic weights
- The equal-abundance assumption is, in practice, always violated, so uniform-weight results represent a lower benchmark rather than a realistic baseline

## Evidence

- [abstract] We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated.: "We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated."
- [discussion] Discussion on cross-habitat vs uniform: "We have demonstrated that this assumption imposes a consistently negative impact on performance, even when compared to deliberately incorrect taxonomic weights"
- [readme] Readme readytowear habitats: [evidence span withheld — anchor status: external_doc]
- [methods] Workflow regression: [evidence span withheld — anchor status: not_found]
