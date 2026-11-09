---
name: amplicon-naive-bayes-classifier-training
description: Use when marker-gene amplicon sequences must be classified to species
  level with a naive Bayes classifier, and the sample's source environment (e.g. an
  EMPO 3 habitat) is known so taxonomic class weights can replace the uniform-weight
  assumption.
when_to_use_negative:
- The query samples' source environment is unknown or lacks sufficient characterization
  for bespoke weight assembly — uniform or average weights may be the only option,
  and cross-habitat weights should not be presented as environment-matched
- The goal is genus-level classification of well-studied references, where existing
  classifiers already achieve ~16% error and bespoke weighting adds little
- Sample distributions are highly unusual (e.g. synthetic populations) without existing
  samples to compile custom bespoke weights from
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3697
tools:
- name: q2-feature-classifier
  role: Trains the multinomial naive Bayes classifier (fit-classifier-naive-bayes,
    scikit-learn MultinomialNB) and classifies queries (classify-sklearn at confidence
    0.7)
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: q2-clawback
  role: Assembles environment-specific taxonomic class weights (generate_class_weights,
    assemble_weights_from_Qiita) used as bespoke class_weight='balanced' during training
  repo: github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Framework hosting the classifier plugin and reference data import
- name: scikit-learn
  role: Provides the MultinomialNB classifier underlying training and classification
- name: imbalanced-learn
  role: Supports class-weighting machinery for the imbalanced reference taxonomic
    distribution
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:6ad81784e78f3e2731ef88981d5d399e53fc67100a1e08ac281e0f182cf26d30
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
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7647058823529411
n_claims: 17
---

# amplicon-naive-bayes-classifier-training

## Summary

Trains a multinomial naive Bayes amplicon taxonomic classifier (q2-feature-classifier fit-classifier-naive-bayes / scikit-learn MultinomialNB) on a reference database such as Greengenes 99% OTUs, optionally with bespoke, environment-specific class weights from q2-clawback to improve species-level accuracy.

## When to use

Use when marker-gene amplicon sequences must be classified to species level with a naive Bayes classifier, and the sample's source environment (e.g. an EMPO 3 habitat) is known so taxonomic class weights can replace the uniform-weight assumption.

## When NOT to use

- The query samples' source environment is unknown or lacks sufficient characterization for bespoke weight assembly — uniform or average weights may be the only option, and cross-habitat weights should not be presented as environment-matched
- The goal is genus-level classification of well-studied references, where existing classifiers already achieve ~16% error and bespoke weighting adds little
- Sample distributions are highly unusual (e.g. synthetic populations) without existing samples to compile custom bespoke weights from

## Inputs

- Greengenes 99% OTU reference sequences and taxonomy (QIIME 2 feature-classifier reference data)
- Held-out query amplicon sequence variant sequences (e.g. Deblur/DADA2 ASVs)
- Environment-specific taxonomic class weights from readytowear or q2-clawback (bespoke weights), plus a uniform-weight baseline

## Outputs

- Trained naive Bayes classifier artifact (.qza) per weighting scheme / habitat
- Species-level taxonomic classifications at confidence 0.7
- Per-habitat species-level error-rate table (e.g. fig1_error_rates.csv) and paired t-test statistics

## How to apply

Import the Greengenes 99% OTU reference sequences and taxonomy into QIIME 2 as feature-classifier reference data. Train the multinomial naive Bayes classifier once with scikit-learn MultinomialNB and class_weight='balanced' bespoke weights (fetched from the readytowear repository for the sample's EMPO 3 habitat, or generated with q2-clawback generate_class_weights / assemble_weights_from_Qiita) and once with uniform weights as baseline. Classify held-out query amplicon sequence variants with classify-sklearn at confidence 0.7 under both weighting schemes, then compute species-level error rates per habitat and compare paired error rates with a paired t-test. Rationale: classification accuracy degrades linearly with how badly the 'all reference species equally likely' assumption is violated, and incorporating environment-specific abundance information cut average species-level error from 25% to 14% across 14 EMPO 3 habitats (paired t-test P = 5.8e-5).

## Related tools

- **q2-feature-classifier** (Trains the multinomial naive Bayes classifier (fit-classifier-naive-bayes, scikit-learn MultinomialNB) and classifies queries (classify-sklearn at confidence 0.7)) — https://github.com/qiime2/q2-feature-classifier/
- **q2-clawback** (Assembles environment-specific taxonomic class weights (generate_class_weights, assemble_weights_from_Qiita) used as bespoke class_weight='balanced' during training) — github.com/BenKaehler/q2-clawback
- **QIIME 2** (Framework hosting the classifier plugin and reference data import)
- **scikit-learn** (Provides the MultinomialNB classifier underlying training and classification)
- **imbalanced-learn** (Supports class-weighting machinery for the imbalanced reference taxonomic distribution)

## Examples

```
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads gg99-ref-seqs.qza --i-reference-taxonomy gg99-ref-tax.qza --i-class-weight bespoke-weights.qza --o-classifier classifier.qza
```

## Evaluation signals

- Species-level average error rate with bespoke weights should drop to roughly 14% vs ~25% with uniform weights across the 14 EMPO 3 habitats
- Paired t-test on per-habitat error rates should give P ≈ 5.8e-5 favoring bespoke weights
- Classifier must be trained once per habitat with class_weight='balanced' and once with uniform weights; classification must use classify-sklearn confidence 0.7
- Bespoke weights should outperform even deliberately incorrect cross-habitat weights for species classification in the majority of cases
- Results table (e.g. fig1_error_rates.csv) should contain paired per-habitat error rates for both schemes for all 14 habitats

## Limitations

- Some degradation in performance for rare species was observed under bespoke weighting
- Bespoke classification is not immune to errors from poorly curated reference data (e.g. reference sequence misannotation) and empirical species distributions can propagate misannotated sample types or sequencing biases into the weights
- Systematic sequencing-technology bias is not controlled unless multiple technologies are integrated for microbial distribution estimation

## Evidence

- [abstract] uniform-weight degradation: "We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated."
- [abstract] error-rate improvement: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [methods] ready-made weights: "If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback, please check out our ready-made weights at [readytowear]"
- [discussion] rare species caveat: "We did observe some degradation in performance for rare species"
- [readme] readytowear citation/primer scope: [evidence span withheld — anchor status: not_found]
