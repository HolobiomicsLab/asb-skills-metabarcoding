---
name: stratified-fold-partitioning-of-reference-sequences
description: Use when evaluating taxonomy-classification accuracy with environment-specific
  (clawback) class weights and you need leak-free, abundance-aware train/test splits
  over reference sequences and empirical samples.
when_to_use_negative:
- You need ready-made weights for a common habitat (e.g., animal-distal-gut, soil-non-saline,
  human-stool EMPO 3 types) rather than benchmarking classification accuracy — use
  the readytowear pre-trained classifiers instead of running the fold workflow
- Your empirical samples lack sufficient characterization to assemble bespoke weights
  — the article recommends average weights from global microbial species distributions
  in that case
- Input is a single unstratified pool of sequences where taxon proportions are not
  estimable per fold — stratification requires enough members per taxon (very small
  taxonomies must be merged into strata)
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3308
tools:
- name: q2-clawback
  role: QIIME 2 plugin providing fetch_Qiita_samples, generate_class_weights, and
    sequence_variants_from_samples used to assemble weights and simulate test samples
    per fold
  repo: https://github.com/BenKaehler/q2-clawback
- name: scikit-learn
  role: Provides StratifiedKFold with shuffle enabled to assign simultaneous stratified
    5-fold splits to reference sequences and empirical samples
- name: imbalanced-learn
  role: Confidence-weighted voting merges small taxonomies lacking sufficient members
    into strata for valid stratification
- name: redbiom
  role: Cache service for rapidly finding and fetching Qiita samples by metadata searches,
    supporting sample discovery for the empirical fold set
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:812ac60899c3ffed90dea33f0f1d1169ec87b28080b78ac8ccc7275e2d9194ca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - imbalanced-learn
  - q2-clawback
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# stratified-fold-partitioning-of-reference-sequences

## Summary

A shuffled stratified 5-fold cross-validation design that simultaneously partitions reference sequences and empirical Qiita samples so every fold carries comparable taxon proportions, while guaranteeing a classifier never sees a test sequence that appeared in its training set and never reuses abundances that informed its taxonomic weights.

## When to use

Use when evaluating taxonomy-classification accuracy with environment-specific (clawback) class weights and you need leak-free, abundance-aware train/test splits over reference sequences and empirical samples.

## When NOT to use

- You need ready-made weights for a common habitat (e.g., animal-distal-gut, soil-non-saline, human-stool EMPO 3 types) rather than benchmarking classification accuracy — use the readytowear pre-trained classifiers instead of running the fold workflow
- Your empirical samples lack sufficient characterization to assemble bespoke weights — the article recommends average weights from global microbial species distributions in that case
- Input is a single unstratified pool of sequences where taxon proportions are not estimable per fold — stratification requires enough members per taxon (very small taxonomies must be merged into strata)

## Inputs

- Empirical sample set fetched from Qiita (via fetch_Qiita_samples) with habitat/sample-type metadata
- Reference sequence database with taxonomy (e.g., Greengenes) sharing the same taxonomy as the samples
- Taxonomic label assignments for reference sequences used as stratification strata

## Outputs

- fold_assignments.csv mapping every reference sequence and sample to its fold per split (exactly five distinct shuffled fold IDs)
- simulation_manifest.csv of simulated test samples generated with sequence_variants_from_samples from held-out test-fold sequences
- Environment-specific class weights from generate_class_weights computed on training folds only

## How to apply

Fetch the empirical sample set with fetch_Qiita_samples and pair it with the reference sequence database (e.g., Greengenes) so both share a common taxonomy. Assign simultaneous shuffled stratified 5-fold splits to reference sequences and empirical samples using scikit-learn's StratifiedKFold with shuffle enabled, so every fold carries comparable taxon proportions. Merge small taxonomies that lack sufficient members for stratification into strata using imbalanced-learn's confidence-weighted voting to decide stratum membership. For each split, call generate_class_weights on the training-fold samples only (holding the test fold out), then simulate test samples from the held-out sequences with sequence_variants_from_samples; the key invariant is that a classifier is never asked to classify a sequence that occurred in its training set and never given abundances that directly contributed to its input taxonomic weights.

## Related tools

- **q2-clawback** (QIIME 2 plugin providing fetch_Qiita_samples, generate_class_weights, and sequence_variants_from_samples used to assemble weights and simulate test samples per fold) — https://github.com/BenKaehler/q2-clawback
- **scikit-learn** (Provides StratifiedKFold with shuffle enabled to assign simultaneous stratified 5-fold splits to reference sequences and empirical samples)
- **imbalanced-learn** (Confidence-weighted voting merges small taxonomies lacking sufficient members into strata for valid stratification)
- **redbiom** (Cache service for rapidly finding and fetching Qiita samples by metadata searches, supporting sample discovery for the empirical fold set) — https://github.com/biocore/redbiom

## Examples

```
from sklearn.model_selection import StratifiedKFold; skf = StratifiedKFold(n_splits=5, shuffle=True); folds = list(skf.split(ref_seqs, ref_taxa))
```

## Evaluation signals

- fold_assignments.csv and simulation_manifest.csv both exist and each contains exactly five distinct shuffled fold IDs
- Every reference sequence and every sample appears in exactly one fold per split (a complete, non-overlapping partition)
- Each fold carries comparable taxon proportions (stratification preserved after small-taxonomy merging)
- No classifier is ever evaluated on a sequence that occurred in its training set, and test abundances never contributed to the input taxonomic weights (leak-free check)
- Class weights for each split were generated only from training-fold samples, verifiable by tracing generate_class_weights calls against the fold-assignment table

## Limitations

- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation), which propagate through the fold evaluation
- Empirical species distributions introduce error sources such as misannotated sample types and sequencing biases propagated into taxonomic weights
- Taxonomies with too few members for stratification cannot be split directly and must be merged into strata, reducing label granularity in those folds

## Evidence

- [methods] The cross-validation strategy was constructed so that a classifier was never asked to classify a sequence that had occurred in its training set, and never generated taxonomic abundances that had directly contributed to its input taxonomic weights.: [evidence span withheld — anchor status: not_found]
- [methods] Assign simultaneous shuffled stratified 5-fold splits to reference sequences and empirical samples using scikit-learn's StratifiedKFold with shuffle enabled so every fold carries comparable taxon proportions.: [evidence span withheld — anchor status: not_found]
- [methods] Merge small taxonomies that lack sufficient members for stratification into strata using imbalanced-learn's confidence-weighted voting to decide stratum membership.: [evidence span withheld — anchor status: not_found]
- [discussion] We demonstrated species-level classification improvement with as few as 122 samples (four-fifths of the saline surface samples from the EMP study; Table 1): "species-level classification improvement with as few as 122 samples (four-fifths of the saline surface samples from the EMP study"
- [readme] If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback, please check out our ready-made weights: "If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback"
