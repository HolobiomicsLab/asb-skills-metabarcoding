---
name: taxonomy-stratum-merging-via-confidence-weighted-voting
description: Use when building shuffled stratified 5-fold splits over reference sequences
  and empirical samples (e.g., Qiita habitats) and some taxonomy labels lack enough
  members to be stratified on their own.
when_to_use_negative:
- All taxonomies have sufficient members for scikit-learn StratifiedKFold stratification,
  so no merging is needed
- You only need off-the-shelf taxonomic weights and can use readytowear pre-trained
  classifiers instead of running the clawback cross-validation workflow
- Your goal is plain k-fold CV that ignores taxon proportions and empirical abundances
  — stratification and stratum merging are unnecessary overhead
edam_topics:
- http://edamontology.org/topic_0699
- http://edamontology.org/topic_3474
tools:
- name: imbalanced-learn
  role: Provides confidence-weighted voting to decide which stratum each under-populated
    taxonomy is merged into
- name: scikit-learn
  role: StratifiedKFold with shuffle enabled produces the simultaneous shuffled 5-fold
    splits over reference sequences and empirical samples that stratum merging must
    support
- name: q2-clawback
  role: Plugin whose abundance-aware classification workflow (generate_class_weights,
    sequence_variants_from_samples, fetch_Qiita_samples) the stratification and merging
    step serves
  repo: github.com/BenKaehler/q2-clawback
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:ca96d5795ce3a25c4786791474c99e4ffadac77ea7e7ec9fdc01ecada2e28ad8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - imbalanced-learn
  - q2-clawback
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0699
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# taxonomy-stratum-merging-via-confidence-weighted-voting

## Summary

Within the empirical-abundance-aware stratified 5-fold cross-validation of q2-clawback, taxonomies with too few reference members to support StratifiedKFold are merged into strata, with stratum membership decided by imbalanced-learn's confidence-weighted voting. This keeps every fold carrying comparable taxon proportions while preserving abundance-aware class weights.

## When to use

Use when building shuffled stratified 5-fold splits over reference sequences and empirical samples (e.g., Qiita habitats) and some taxonomy labels lack enough members to be stratified on their own.

## When NOT to use

- All taxonomies have sufficient members for scikit-learn StratifiedKFold stratification, so no merging is needed
- You only need off-the-shelf taxonomic weights and can use readytowear pre-trained classifiers instead of running the clawback cross-validation workflow
- Your goal is plain k-fold CV that ignores taxon proportions and empirical abundances — stratification and stratum merging are unnecessary overhead

## Inputs

- Reference sequence database paired with a common taxonomy (e.g., Greengenes or RefSeq)
- Empirical sample set fetched from Qiita via fetch_Qiita_samples
- Taxon membership counts per taxonomy label (for identifying under-populated taxonomies)

## Outputs

- Stratum assignment table mapping small taxonomies to merged strata
- fold_assignments.csv mapping every reference sequence and sample to one of five shuffled folds per split
- simulation_manifest.csv recording simulated test samples generated with sequence_variants_from_samples

## How to apply

After pairing the Qiita sample set with the reference sequence database under a shared taxonomy (e.g., Greengenes or RefSeq), assign simultaneous shuffled StratifiedKFold (n_splits=5, shuffle=True) splits to reference sequences and empirical samples. Identify taxonomies whose member counts are too small for stratification, then merge them into strata using imbalanced-learn's confidence-weighted voting to decide which stratum each small taxonomy joins. Proceed per split by calling generate_class_weights on the training-fold samples only (test fold held out) and simulate test samples with sequence_variants_from_samples. The rationale is that the naive Bayes classifier's accuracy degrades when the equal-observation-probability assumption is violated, so folds must retain realistic taxon proportions without leaking test taxa or abundances into training.

## Related tools

- **imbalanced-learn** (Provides confidence-weighted voting to decide which stratum each under-populated taxonomy is merged into)
- **scikit-learn** (StratifiedKFold with shuffle enabled produces the simultaneous shuffled 5-fold splits over reference sequences and empirical samples that stratum merging must support)
- **q2-clawback** (Plugin whose abundance-aware classification workflow (generate_class_weights, sequence_variants_from_samples, fetch_Qiita_samples) the stratification and merging step serves) — github.com/BenKaehler/q2-clawback

## Examples

```
from sklearn.model_selection import StratifiedKFold; skf = StratifiedKFold(n_splits=5, shuffle=True); splits = skf.split(reference_seqs, stratum_labels)
```

## Evaluation signals

- fold_assignments.csv and simulation_manifest.csv both exist after the run
- Each output file contains exactly five distinct shuffled fold IDs per split
- Every reference sequence and every empirical sample appears in exactly one fold per split (no duplicates, no omissions)
- No test-fold sequence or its abundance contributed to the training set or to the class weights used to classify it
- Each fold carries comparable taxon proportions, with previously under-populated taxonomies represented through their merged strata

## Limitations

- Merging is a compromise: small taxonomies are not stratified individually, and the article observed some degradation in performance for rare species under bespoke weighting
- The approach inherits errors from empirical species distributions, such as misannotation of sample types or sequencing biases propagated into taxonomic weights
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation)

## Evidence

- [methods] Merge small taxonomies that lack sufficient members for stratification into strata using imbalanced-learn's confidence-weighted voting to decide stratum membership.: [evidence span withheld — anchor status: not_found]
- [methods] Assign simultaneous shuffled stratified 5-fold splits to reference sequences and empirical samples using scikit-learn's StratifiedKFold with shuffle enabled so every fold carries comparable taxon proportions.: [evidence span withheld — anchor status: not_found]
- [abstract] Imbalanced-learn: A python toolbox to tackle the curse of imbalanced datasets in machine learning: "Imbalanced-learn: A python toolbox to tackle the curse of imbalanced datasets in machine learning"
- [discussion] We did observe some degradation in performance for rare species: "We did observe some degradation in performance for rare species"
