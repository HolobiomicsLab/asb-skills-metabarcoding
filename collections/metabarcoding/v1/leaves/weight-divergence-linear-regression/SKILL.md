---
name: weight-divergence-linear-regression
description: 'Use when evaluating whether taxonomic class weights assembled for one
  environment remain useful for samples from another: you have per-habitat q2-clawback
  weight sets, per-combination species-level error rates, and need to model error
  rate as a linear function of weight divergence.'
when_to_use_negative:
- The query samples' source habitat has bespoke q2-clawback weights available — use
  those directly instead of testing cross-habitat transfer
- You only need genus-level classification accuracy; the article's species-level error-rate
  gains (25% to 14%) are the target of this regression, not genus-level behaviour
- You only need to assemble weights (fetch_Qiita_samples / generate_class_weights)
  without measuring how weight mismatch degrades accuracy
edam_operation: http://edamontology.org/operation_0366
edam_topics:
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_0622
tools:
- name: q2-clawback
  role: assembles the per-habitat taxonomic class weights (and cross-habitat weight
    sets) whose divergence from source-habitat weights is regressed against error
    rate
  repo: github.com/BenKaehler/q2-clawback
- name: q2-feature-classifier
  role: trains the scikit-learn multinomial naive Bayes classifier (class weights
    enabled via imbalanced-learn conventions) used to produce species-level classifications
    under each weight set
- name: scikit-learn
  role: fits the linear regression of species-level error rate on weight divergence
    used to recover the reported r2 = 0.57
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:f0386b7ab8bba83e2803a2f3b0493c7cc0e734103374c338fe784f3901a9400f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0366
  edam_topics:
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# weight-divergence-linear-regression

## Summary

Quantifies how naive Bayes species-level classification accuracy degrades as a function of the divergence between applied taxonomic class weights (e.g. cross-habitat q2-clawback weights) and the source habitat's bespoke weights. Establishes that classification error grows linearly with weight divergence, so mis-matched weights can be predicted to hurt in a quantifiable way.

## When to use

Use when evaluating whether taxonomic class weights assembled for one environment remain useful for samples from another: you have per-habitat q2-clawback weight sets, per-combination species-level error rates, and need to model error rate as a linear function of weight divergence.

## When NOT to use

- The query samples' source habitat has bespoke q2-clawback weights available — use those directly instead of testing cross-habitat transfer
- You only need genus-level classification accuracy; the article's species-level error-rate gains (25% to 14%) are the target of this regression, not genus-level behaviour
- You only need to assemble weights (fetch_Qiita_samples / generate_class_weights) without measuring how weight mismatch degrades accuracy

## Inputs

- EMPO 3 habitat sample sequence variants (e.g. Deblur or DADA2 feature tables) for 14 habitats
- Greengenes reference sequences and taxonomy files
- q2-clawback class weights per habitat plus a uniform-weights control set
- Species-level classification error rates for every habitat x weight-set combination

## Outputs

- Comparison table of 182 cross-habitat-versus-uniform species-level error-rate comparisons (figures_3_4_results.csv)
- Win tally of cross-habitat vs uniform weights (expected 117/182)
- Linear regression coefficients and Pearson correlation (expected r2 = 0.57, P < 2.2e-16)
- Fig. 3-4 equivalent PNG plots of error rate vs weight divergence

## How to apply

Train a naive Bayes classifier with QIIME 2's q2-feature-classifier (scikit-learn multinomial naive Bayes, class weights enabled via imbalanced-learn conventions) on Greengenes reference sequences/taxonomy, then classify each of the 14 EMPO 3 habitats' samples using class weights assembled by q2-clawback from each of the other 13 habitats, plus a uniform-weights control, yielding 182 cross-habitat-versus-uniform comparisons. For every habitat x weight-set combination, compute the species-level classification error rate and the divergence between the applied weights and the source habitat's bespoke weights. Tally where cross-habitat weights beat uniform weights (expected: 117 of 182), fit a linear regression of species-level error rate on weight divergence with scikit-learn, and compute the Pearson correlation (expected r2 = 0.57, P < 2.2e-16). The linear fit is the point: the article's core claim is that accuracy degrades linearly as the equal-abundance assumption is violated, so error rate on weight divergence is the correct model rather than a nonlinear one.

## Related tools

- **q2-clawback** (assembles the per-habitat taxonomic class weights (and cross-habitat weight sets) whose divergence from source-habitat weights is regressed against error rate) — github.com/BenKaehler/q2-clawback
- **q2-feature-classifier** (trains the scikit-learn multinomial naive Bayes classifier (class weights enabled via imbalanced-learn conventions) used to produce species-level classifications under each weight set)
- **scikit-learn** (fits the linear regression of species-level error rate on weight divergence used to recover the reported r2 = 0.57)

## Examples

```
from sklearn.linear_model import LinearRegression; from scipy.stats import pearsonr; m = LinearRegression().fit(divergence.reshape(-1,1), err); r2, p = pearsonr(divergence, err)
```

## Evaluation signals

- Cross-habitat weights outperform uniform weights in the expected majority of comparisons: verify 117 of 182
- Pearson correlation of species-level error rate on weight divergence recovers r2 = 0.57 with P < 2.2e-16
- The error-rate vs divergence relationship is consistent with a linear fit (per the article's headline claim), not merely monotone
- The comparison table contains exactly 182 rows (14 habitats x 13 cross-habitat weight sets), and uniform-weights control runs are included for each habitat
- figures_3_4_results.csv contains the comparison table, win tally, and regression statistics, and PNG plots render without missing habitat/weight-set entries

## Limitations

- Performance degradation was observed for rare species under bespoke weighting, so the linear error-divergence trend may not hold uniformly across abundance classes
- Empirical species distributions used to build weights introduce error sources such as misannotation of sample types and sequencing biases, which can distort the divergence axis
- Bespoke classification is not immune to errors resulting from poorly curated reference data (e.g. reference sequence misannotation in Greengenes)

## Evidence

- [abstract] Linear degradation claim: "We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated."
- [discussion] Cross-habitat vs uniform majority: "these weights still outperformed uniform weights for species classification in the majority of cases"
- [discussion] Specificity improves accuracy: [evidence span withheld — anchor status: not_found]
- [discussion] Uniform assumption negative impact: "We have demonstrated that this assumption imposes a consistently negative impact on performance, even when compared to deliberately incorrect taxonomic weights"
- [readme] Ready-made weights provenance: [evidence span withheld — anchor status: not_found]
