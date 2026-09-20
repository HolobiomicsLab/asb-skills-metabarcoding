---
name: qiime2-artifact-workflow
description: Use when classifying 16S amplicon sequence variants against Greengenes
  99% (or Silva 138.1) reference data in QIIME 2 and you want environment-specific
  (e.g., EMPO 3 habitat) taxonomic class weights rather than the uniform-weight assumption.
when_to_use_negative:
- Query data are shotgun metagenomes or non-marker-gene sequences rather than 16S
  amplicon sequences, since the classifier and class weights were trained on marker-gene
  amplicon reference data
- The sample's source environment is highly unusual (e.g., a synthetic population)
  with no characterized habitat weights — the article recommends compiling custom
  bespoke weights from existing samples instead of relying on stock artifacts
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3316
tools:
- name: QIIME 2
  role: Framework providing artifact/semantic-type infrastructure for importing reference
    data, training classifiers, and classifying ASVs
  repo: https://qiime2.org
- name: q2-feature-classifier
  role: Trains fit-classifier-naive-bayes (scikit-learn MultinomialNB) with bespoke
    or uniform class weights and runs classify-sklearn at confidence 0.7
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: q2-clawback
  role: Assembles taxonomic class weights from sample data; ready-made weights are
    published in readytowear for common habitats
  repo: https://github.com/BenKaehler/q2-clawback
- name: redbiom
  role: Searches and fetches sample data and metadata (e.g., by metadata category
    or contained features) used to assemble class weights
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:018000b8ef290dc60dfe0a9e460b9934ec47f300aff5457c9d0d0efbd443f1b1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# qiime2-artifact-workflow

## Summary

A QIIME 2 semantic-type-driven pipeline for marker-gene taxonomic classification: reference sequences/taxonomy are imported as artifacts, a multinomial naive Bayes classifier is trained per environment with bespoke class weights, and held-out amplicon sequence variants are classified at confidence 0.7.

## When to use

Use when classifying 16S amplicon sequence variants against Greengenes 99% (or Silva 138.1) reference data in QIIME 2 and you want environment-specific (e.g., EMPO 3 habitat) taxonomic class weights rather than the uniform-weight assumption.

## When NOT to use

- Query data are shotgun metagenomes or non-marker-gene sequences rather than 16S amplicon sequences, since the classifier and class weights were trained on marker-gene amplicon reference data
- The sample's source environment is highly unusual (e.g., a synthetic population) with no characterized habitat weights — the article recommends compiling custom bespoke weights from existing samples instead of relying on stock artifacts

## Inputs

- Greengenes 99% OTU reference sequences and taxonomy imported as QIIME 2 feature-classifier artifacts (.qza)
- EMPO 3 habitat-specific taxonomic class weights from the readytowear repository
- Uniform-weight baseline class weights
- Held-out query amplicon sequence variants (ASVs) per EMPO 3 habitat

## Outputs

- Trained naive Bayes classifier artifacts per weighting scheme (.qza)
- Per-habitat species-level taxonomy classifications at confidence 0.7
- Per-habitat and overall average species-level error rates (fig1_error_rates.csv)

## How to apply

Import the Greengenes 99% OTU reference sequences and taxonomy into QIIME 2 as feature-classifier reference artifacts. Fetch ready-made EMPO 3 habitat-specific taxonomic class weights from the readytowear repository (or assemble them with q2-clawback) and prepare a uniform-weight baseline. Train the q2-feature-classifier fit-classifier-naive-bayes classifier (scikit-learn MultinomialNB, class_weight='balanced') on the Greengenes 99% reference once per habitat under each weighting scheme. Classify each habitat's held-out query amplicon sequence variants with classify-sklearn at confidence 0.7, then compare paired per-habitat species-level error rates between the bespoke and uniform schemes to confirm the expected improvement (14% vs 25%, paired t-test P = 5.8e-5).

## Related tools

- **QIIME 2** (Framework providing artifact/semantic-type infrastructure for importing reference data, training classifiers, and classifying ASVs) — https://qiime2.org
- **q2-feature-classifier** (Trains fit-classifier-naive-bayes (scikit-learn MultinomialNB) with bespoke or uniform class weights and runs classify-sklearn at confidence 0.7) — https://github.com/qiime2/q2-feature-classifier/
- **q2-clawback** (Assembles taxonomic class weights from sample data; ready-made weights are published in readytowear for common habitats) — https://github.com/BenKaehler/q2-clawback
- **redbiom** (Searches and fetches sample data and metadata (e.g., by metadata category or contained features) used to assemble class weights) — https://github.com/biocore/redbiom

## Examples

```
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads gg99-ref-seqs.qza --i-reference-taxonomy gg99-tax.qza --i-class-weight empo3-habitat-weights.qza --o-classifier bespoke-classifier.qza
```

## Evaluation signals

- Classifier artifacts pass QIIME 2 semantic-type validation as FeatureClassifier artifacts
- Species-level average error rate for bespoke weighting falls to ~14% vs ~25% for uniform weighting across the 14 EMPO 3 habitats
- Paired t-test over per-habitat error rates yields P ≈ 5.8e-5
- fig1_error_rates.csv contains one paired error-rate row per EMPO 3 habitat and reproduces the reported per-habitat and overall averages
- Classification runs used confidence 0.7 for both weighting schemes, so error-rate differences are attributable to weights alone

## Limitations

- Classification accuracy degrades linearly as the uniform-weight assumption is violated, and the assumption is in practice always violated
- Performance degrades for rare species under bespoke weighting
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., misannotated reference sequences) or misannotated sample types and sequencing biases propagated into weights
- Natural sample types lacking sufficient characterization cannot have bespoke weights assembled; global average weights are then recommended over uniform weights

## Evidence

- [readme] Ready-made taxonomic weights generated by q2-clawback for use with the q2-feature-classifier taxonomy classifier.: [evidence span withheld — anchor status: external_doc]
- [readme] They were all trained using Silva 138.1 using both full-length and 515f-806r trimmed reads.: [evidence span withheld — anchor status: external_doc]
- [abstract] At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%).: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [abstract] By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types.: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [other] Classify each habitat's held-out query amplicon sequence variants with classify-sklearn at confidence 0.7 under both weighting schemes.: [evidence span withheld — anchor status: not_found]
