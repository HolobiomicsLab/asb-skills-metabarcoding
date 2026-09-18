---
name: empirical-test-sample-simulation-from-reference-sequences
description: Use when evaluating a taxonomy classifier's accuracy under environment-specific
  class weights and you need simulated test samples drawn only from sequences absent
  from the classifier's training fold.
when_to_use_negative:
- You have real held-out amplicon samples with verified taxonomic labels, so simulation
  is unnecessary
- The reference sequences were already included in classifier training or their abundances
  contributed to the class weights (this would break the paper's non-contamination
  guarantee)
- You only need ready-made taxonomic weights for a common habitat without measuring
  classifier accuracy — use readytowear weights instead
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3308
tools:
- name: q2-clawback
  role: QIIME 2 plugin providing sequence_variants_from_samples and generate_class_weights
    for simulating test samples and assembling empirical taxonomic weights
  repo: github.com/BenKaehler/q2-clawback
- name: scikit-learn
  role: StratifiedKFold with shuffle enabled to produce stratified 5-fold splits of
    reference sequences and empirical samples
- name: imbalanced-learn
  role: Confidence-weighted voting to merge small taxonomies lacking sufficient members
    for stratification into strata
- name: redbiom
  role: Backend for fetching empirical sample data and metadata from Qiita (fetch_Qiita_samples)
  repo: biocore/redbiom
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:ab4490a354dcd6cd3f3cc028541c61f882b02782cf8149a2b3c8cb589d23e5d9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - imbalanced-learn
  - q2-clawback
  - redbiom
  - scikit-learn
  - sequence_variants_from_samples
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# Empirical test-sample simulation from reference sequences

## Summary

Simulates artificial test samples by drawing sequence variants from held-out test-fold reference sequences, with abundances informed by empirical taxonomic weights, so that classifier accuracy can be scored against known labels without contaminating the training set.

## When to use

Use when evaluating a taxonomy classifier's accuracy under environment-specific class weights and you need simulated test samples drawn only from sequences absent from the classifier's training fold.

## When NOT to use

- You have real held-out amplicon samples with verified taxonomic labels, so simulation is unnecessary
- The reference sequences were already included in classifier training or their abundances contributed to the class weights (this would break the paper's non-contamination guarantee)
- You only need ready-made taxonomic weights for a common habitat without measuring classifier accuracy — use readytowear weights instead

## Inputs

- Held-out test-fold reference sequences from the Greengenes (or RefSeq) reference database sharing a common taxonomy with the empirical samples
- Empirical sample set fetched from Qiita (via fetch_Qiita_samples) for training-fold class-weight generation
- Environment-specific class weights generated from training-fold samples
- Fold-assignment table mapping sequences and samples to folds

## Outputs

- Simulated test samples with sequence variants and known ground-truth taxonomic labels (for scoring classification accuracy)
- simulation_manifest.csv recording the simulated test samples per split
- fold_assignments.csv mapping every sequence and sample to its fold

## How to apply

After assigning simultaneous shuffled stratified 5-fold splits to both the reference sequence database and the empirical sample set (via scikit-learn's StratifiedKFold with shuffle=True), generate environment-specific class weights with generate_class_weights on the training-fold samples only. Then use sequence_variants_from_samples to simulate test samples from the held-out test-fold sequences, so the simulated abundances never contributed to the input taxonomic weights and no classified sequence appeared in the training set. This construction mirrors the paper's design where 'a classifier was never asked to classify a sequence that had occurred in its training set, and never generated taxonomic abundances that had directly contributed to its input taxonomic weights.' Score species-level classification accuracy against the known labels of the simulated variants. Merge small taxonomies lacking sufficient members for stratification into strata using imbalanced-learn's confidence-weighted voting before splitting.

## Related tools

- **q2-clawback** (QIIME 2 plugin providing sequence_variants_from_samples and generate_class_weights for simulating test samples and assembling empirical taxonomic weights) — github.com/BenKaehler/q2-clawback
- **scikit-learn** (StratifiedKFold with shuffle enabled to produce stratified 5-fold splits of reference sequences and empirical samples)
- **imbalanced-learn** (Confidence-weighted voting to merge small taxonomies lacking sufficient members for stratification into strata)
- **redbiom** (Backend for fetching empirical sample data and metadata from Qiita (fetch_Qiita_samples)) — biocore/redbiom

## Examples

```
from q2_clawback import sequence_variants_from_samples; simulated = sequence_variants_from_samples(test_fold_seqs, class_weights, n=1000)
```

## Evaluation signals

- Both fold_assignments.csv and simulation_manifest.csv exist after the workflow completes
- fold_assignments.csv contains exactly five distinct shuffled fold IDs per split
- Every reference sequence and empirical sample appears in exactly one fold per split
- No simulated test sequence occurred in the classifier's training set, and no simulated abundance contributed to the input taxonomic weights
- Each fold carries comparable taxon proportions (stratification preserved)

## Limitations

- Simulated samples inherit biases from the reference database; bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation)
- Empirical species distributions can propagate misannotated sample types and sequencing biases into the taxonomic weights used to simulate abundances
- Systematic errors from bias in common sequencing techniques are not controlled without integrating multiple technologies for microbial distribution estimation
- Performance degradation was observed for rare species under bespoke weighting

## Evidence

- [methods] non-contamination design: [evidence span withheld — anchor status: not_found]
- [methods] stratified fold assignment: [evidence span withheld — anchor status: not_found]
- [methods] test-sample simulation step: [evidence span withheld — anchor status: not_found]
- [abstract] plugin scope: "q2-clawback provides a straightforward alternative for samples from common environments."
- [readme] citation context: [evidence span withheld — anchor status: external_doc]
