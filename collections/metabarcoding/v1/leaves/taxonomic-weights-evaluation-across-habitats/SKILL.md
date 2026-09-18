---
name: taxonomic-weights-evaluation-across-habitats
description: Use when you need to quantify how far taxonomic weights can diverge from
  a sample's true source environment before species-level classification accuracy
  suffers, or to justify uniform vs weighted classifiers.
when_to_use_negative:
- You have correctly matched source-habitat (bespoke) weights available — the paper
  shows the more specific the weights are to the query sample's environment, the better
  the accuracy, so bespoke weights should be used rather than benchmarked cross-habitat
  substitutes
- Your sample comes from a natural environment lacking sufficient characterization
  for bespoke weight assembly and you plan to use average global-distribution weights
  — this skill is an evaluation harness, not a weight-assembly method
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0637
tools:
- name: q2-clawback
  role: Assembles the cross-habitat taxonomic class weights applied to the naive Bayes
    classifier
  repo: github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Framework hosting the feature-classifier plugin used to train and run the
    weighted naive Bayes taxonomy classifier
  repo: https://qiime2.org
- name: q2-feature-classifier
  role: Trains the scikit-learn multinomial naive Bayes classifier with class weights
    and performs the species-level classifications
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: scikit-learn
  role: Provides the multinomial naive Bayes classifier and the linear regression
    fit of error rate on weight divergence
- name: imbalanced-learn
  role: Conventions used to enable class weights in the imbalanced classification
    setting
- name: redbiom
  role: Sample discovery/retrieval system used to fetch Qiita/EMP sample data underlying
    the weights and test samples
  repo: biocore/redbiom
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:4bf934a96c56e712348e3468c91d077fed97c8a04e3b8bfae0cba223cf782e64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# taxonomic-weights-evaluation-across-habitats

## Summary

Benchmark of how taxonomic class weights assembled from non-matching habitats (cross-habitat weights) degrade naive Bayes species-level classification, relative to bespoke source-habitat and uniform weights, across the 14 EMPO 3 habitats.

## When to use

Use when you need to quantify how far taxonomic weights can diverge from a sample's true source environment before species-level classification accuracy suffers, or to justify uniform vs weighted classifiers.

## When NOT to use

- You have correctly matched source-habitat (bespoke) weights available — the paper shows the more specific the weights are to the query sample's environment, the better the accuracy, so bespoke weights should be used rather than benchmarked cross-habitat substitutes
- Your sample comes from a natural environment lacking sufficient characterization for bespoke weight assembly and you plan to use average global-distribution weights — this skill is an evaluation harness, not a weight-assembly method

## Inputs

- EMPO 3 habitat sample sequence variants (feature tables) for 14 habitats
- Greengenes reference sequences and taxonomy
- q2-clawback class-weights artifacts assembled from each of the 13 non-source habitats
- Uniform-weights control weight set

## Outputs

- figures_3_4_results.csv containing the 182-comparison table, win tally (117/182), and linear-regression/Pearson statistics (r2 = 0.57, P < 2.2e-16)
- PNG plots reproducing the Fig. 3-4 equivalents (error rate vs weight divergence)
- Species-level classification error rates for every habitat x weight-set combination

## How to apply

Train a QIIME 2 naive Bayes classifier (q2-feature-classifier, scikit-learn multinomial NB with class weights enabled via imbalanced-learn) on Greengenes reference sequences/taxonomy. For each of the 14 EMPO 3 habitats, classify its samples using q2-clawback weights assembled from each of the other 13 habitats, plus a uniform-weights control, then compute species-level error rates for every habitat x weight-set combination (182 comparisons). Tally the comparisons where cross-habitat weights beat uniform weights (expected 117 of 182), and fit a linear regression of species-level error rate on weight divergence with scikit-learn, computing the Pearson correlation (expected r2 = 0.57, P < 2.2e-16). The rationale is that classification accuracy degrades linearly with the degree to which the equal-abundance assumption is violated, so even deliberately incorrect weights should be benchmarked against uniform weights as a control. Write the comparison table, win tally, and regression statistics to figures_3_4_results.csv and render Fig. 3-4 equivalents as PNG plots.

## Related tools

- **q2-clawback** (Assembles the cross-habitat taxonomic class weights applied to the naive Bayes classifier) — github.com/BenKaehler/q2-clawback
- **QIIME 2** (Framework hosting the feature-classifier plugin used to train and run the weighted naive Bayes taxonomy classifier) — https://qiime2.org
- **q2-feature-classifier** (Trains the scikit-learn multinomial naive Bayes classifier with class weights and performs the species-level classifications) — https://github.com/qiime2/q2-feature-classifier/
- **scikit-learn** (Provides the multinomial naive Bayes classifier and the linear regression fit of error rate on weight divergence)
- **imbalanced-learn** (Conventions used to enable class weights in the imbalanced classification setting)
- **redbiom** (Sample discovery/retrieval system used to fetch Qiita/EMP sample data underlying the weights and test samples) — biocore/redbiom

## Examples

```
from scipy.stats import pearsonr; r, p = pearsonr(weight_divergence, species_error_rate); assert abs(r*r - 0.57) < 0.01 and p < 2.2e-16
```

## Evaluation signals

- Comparison count invariant: 14 habitats x 13 cross-habitat weight sets + uniform control yields exactly 182 comparisons per habitat pairing scheme
- Win tally: cross-habitat weights outperform uniform weights in 117 of 182 species-classification comparisons
- Regression recovery: linear fit of species-level error rate on weight divergence gives Pearson r2 = 0.57 with P < 2.2e-16
- Monotonicity check: cross-habitat classification error rate increases in proportion to divergence between applied weights and the source habitat's bespoke weights
- figures_3_4_results.csv contains the full comparison table, win tally, and regression statistics, and the PNG plots visually match the Fig. 3-4 linear trend

## Limitations

- Some degradation in performance for rare species was observed under bespoke weighting, so error-rate trends may not hold uniformly across abundance classes
- Even correct-habitat weighting cannot fix errors from poorly curated reference data (e.g., reference sequence misannotation), which propagate into the weights
- Empirical species distributions introduce error sources such as misannotated sample types and sequencing biases; systematic sequencing-technology bias is not controlled without integrating multiple technologies

## Evidence

- [discussion] Cross-habitat weights still beat uniform: "We have demonstrated that this assumption imposes a consistently negative impact on performance, even when compared to deliberately incorrect taxonomic weights"
- [discussion] Majority win over uniform: "these weights still outperformed uniform weights for species classification in the majority of cases"
- [discussion] Specificity improves accuracy: "demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy"
- [abstract] Linear degradation with assumption violation: "We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated."
- [readme] Ready-made weights provenance: [evidence span withheld — anchor status: external_doc]
