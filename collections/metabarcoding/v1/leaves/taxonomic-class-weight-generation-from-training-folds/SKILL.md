---
name: taxonomic-class-weight-generation-from-training-folds
description: Use when evaluating taxonomic classifiers (e.g. naive Bayes in q2-feature-classifier)
  with empirical taxonomic weights and you must guarantee that no test-fold sequence
  or abundance contributed to the class weights used for classification.
when_to_use_negative:
- Ready-made weights for the habitat already exist in readytowear (e.g. animal-distal-gut,
  soil-non-saline, human-oral, human-stool EMPO 3 types) — use those instead of regenerating
  weights
- The query sample's environment lacks sufficient characterization for bespoke weight
  assembly; average weights estimated from global microbial species distributions
  are superior to uniform weights there
- Input is already a set of assembled taxonomic weights and the goal is classification,
  not weight-assembly evaluation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: generate_class_weights
  role: Forms environment-specific class weights from training-fold samples while
    the test fold is held out
  repo: https://github.com/BenKaehler/q2-clawback
- name: fetch_Qiita_samples
  role: Fetches the empirical sample set whose abundances feed weight generation,
    paired with the reference database
  repo: https://github.com/BenKaehler/q2-clawback
- name: sequence_variants_from_samples
  role: Simulates test samples from held-out test-fold sequences so classification
    accuracy can be scored against known labels
  repo: https://github.com/BenKaehler/q2-clawback
- name: scikit-learn
  role: Provides shuffled StratifiedKFold 5-fold splits of reference sequences and
    empirical samples
- name: imbalanced-learn
  role: Handles small taxonomies that lack sufficient members for stratification by
    merging them into strata via confidence-weighted voting
- name: q2-clawback
  role: Plugin providing the clawback weight-assembly machinery underpinning class-weight
    generation
  repo: https://github.com/BenKaehler/q2-clawback
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:66b34832e7e7da4868a2ba22642e824afe278f73990cc45e1def6fe0bf1a3f8f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - fetch_Qiita_samples
  - generate_class_weights
  - imbalanced-learn
  - q2-clawback
  - scikit-learn
  - sequence_variants_from_samples
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# taxonomic-class-weight-generation-from-training-folds

## Summary

Generates environment-specific taxonomic class weights for naive Bayes sequence classification by running generate_class_weights only on training-fold samples of a shuffled stratified 5-fold cross-validation, so that held-out sequences and abundances never leak into the weights used to classify them.

## When to use

Use when evaluating taxonomic classifiers (e.g. naive Bayes in q2-feature-classifier) with empirical taxonomic weights and you must guarantee that no test-fold sequence or abundance contributed to the class weights used for classification.

## When NOT to use

- Ready-made weights for the habitat already exist in readytowear (e.g. animal-distal-gut, soil-non-saline, human-oral, human-stool EMPO 3 types) — use those instead of regenerating weights
- The query sample's environment lacks sufficient characterization for bespoke weight assembly; average weights estimated from global microbial species distributions are superior to uniform weights there
- Input is already a set of assembled taxonomic weights and the goal is classification, not weight-assembly evaluation

## Inputs

- Empirical sample set fetched from Qiita (via fetch_Qiita_samples, redbiom context)
- Reference sequence database sharing a common taxonomy with the samples (e.g. Greengenes)
- Sample metadata category defining strata/habitat (e.g. EMPO 3 habitat types)
- sklearn StratifiedKFold fold assignments over sequences and samples

## Outputs

- Environment-specific taxonomic class weights per training fold (generate_class_weights output)
- fold_assignments.csv mapping every sequence and sample to its fold per split
- simulation_manifest.csv recording simulated test samples from held-out sequences
- Per-fold classification accuracy against known labels

## How to apply

Pair the empirical sample set (fetched from Qiita via fetch_Qiita_samples) with the reference sequence database so both share a common taxonomy (e.g. Greengenes). Assign simultaneous shuffled stratified 5-fold splits to reference sequences and empirical samples using scikit-learn's StratifiedKFold with shuffle enabled, so each fold carries comparable taxon proportions; merge small taxonomies lacking enough members for stratification into strata using imbalanced-learn's confidence-weighted voting. For each split, call generate_class_weights on the training-fold samples only, holding the test fold out, then simulate test samples from held-out sequences with sequence_variants_from_samples so accuracy can be scored against known labels. The rationale is the paper's core guarantee: a classifier is never asked to classify a sequence that occurred in its training set, and never generates taxonomic abundances that directly contributed to its input weights.

## Related tools

- **generate_class_weights** (Forms environment-specific class weights from training-fold samples while the test fold is held out) — https://github.com/BenKaehler/q2-clawback
- **fetch_Qiita_samples** (Fetches the empirical sample set whose abundances feed weight generation, paired with the reference database) — https://github.com/BenKaehler/q2-clawback
- **sequence_variants_from_samples** (Simulates test samples from held-out test-fold sequences so classification accuracy can be scored against known labels) — https://github.com/BenKaehler/q2-clawback
- **scikit-learn** (Provides shuffled StratifiedKFold 5-fold splits of reference sequences and empirical samples)
- **imbalanced-learn** (Handles small taxonomies that lack sufficient members for stratification by merging them into strata via confidence-weighted voting)
- **q2-clawback** (Plugin providing the clawback weight-assembly machinery underpinning class-weight generation) — https://github.com/BenKaehler/q2-clawback

## Examples

```
from sklearn.model_selection import StratifiedKFold; skf = StratifiedKFold(n_splits=5, shuffle=True); weights = generate_class_weights(train_samples, taxonomy, reference_seqs)
```

## Evaluation signals

- fold_assignments.csv and simulation_manifest.csv both exist and contain exactly five distinct shuffled fold IDs
- Every reference sequence and sample appears in exactly one fold per split
- No test-fold sequence occurs in the training set used by the classifier (leakage check)
- Class weights for a fold are generated only from training-fold samples, never from held-out samples' abundances
- Simulated test samples carry known labels allowing per-fold accuracy scoring

## Limitations

- Performance degradation was observed for rare species under bespoke weighting
- Empirical species distributions introduce potential error sources such as misannotated sample types and sequencing biases propagated into taxonomic weights
- Bespoke classification is not immune to errors from poorly curated reference data (e.g. reference sequence misannotation)
- Small taxonomies without sufficient members cannot be stratified directly and must be merged into strata

## Evidence

- [methods] a classifier was never asked to classify a sequence that had occurred in its training set, and never generated taxonomic abundances that had directly contributed to its input taxonomic weights: [evidence span withheld — anchor status: not_found]
- [methods] Assign simultaneous shuffled stratified 5-fold splits to reference sequences and empirical samples using scikit-learn's StratifiedKFold with shuffle enabled: [evidence span withheld — anchor status: not_found]
- [abstract] By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types.: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [discussion] demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy: "demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy"
- [readme] Ready-made taxonomic weights generated by q2-clawback for use with the q2-feature-classifier taxonomy classifier.: [evidence span withheld — anchor status: external_doc]
