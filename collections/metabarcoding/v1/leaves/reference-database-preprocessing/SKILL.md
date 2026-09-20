---
name: reference-database-preprocessing
description: Use when building taxonomic classifiers for amplicon ASV/OTU queries
  where species-level accuracy depends on reference class priors — e.g., importing
  Greengenes 99% reference data into QIIME 2 and assembling EMPO 3 habitat-specific
  or uniform class_weight inputs for fit-classifier-naive-bayes.
when_to_use_negative:
- Ready-made pretrained classifiers for the sample habitat (e.g., animal-distal-gut,
  soil-non-saline, human-stool) already exist on Zenodo — download them instead of
  retraining
- The query data are metagenomic profiles (MetaPhlAn2-style) rather than 16S marker-gene
  amplicon sequences, since clawback weights are built from marker-gene sample distributions
- The sample environment lacks sufficient characterization to assemble bespoke weights
  and no matching readytowear weights exist — use global average weights instead of
  forcing habitat-specific preparation
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: framework hosting reference data artifacts and the feature-classifier plugin
    used to import and train classifiers
  repo: https://qiime2.org
- name: q2-feature-classifier
  role: fit-classifier-naive-bayes trains the MultinomialNB classifier on Greengenes
    99% references with class_weight='balanced'; classify-sklearn assigns taxonomy
    at confidence 0.7
- name: q2-clawback
  role: assembles environment-specific taxonomic class weights from Qiita samples
    when ready-made weights are not used
  repo: https://github.com/BenKaehler/q2-clawback
- name: readytowear
  role: supplies ready-made EMPO 3 habitat taxonomic weights so clawback itself need
    not be run
  repo: https://github.com/BenKaehler/readytowear
- name: redbiom
  role: rapid sample discovery and feature characterization used to find and fetch
    Qiita samples underlying the weights
  repo: https://github.com/biocore/redbiom
- name: scikit-learn
  role: provides the MultinomialNB estimator underlying fit-classifier-naive-bayes
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:36784c289225f878cc8e16e7b184636b22bec0b56ab1f0cc01af2826ddc92c61
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - readytowear
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# reference-database-preprocessing

## Summary

Prepare reference sequence/taxonomy data (e.g., Greengenes 99% OTUs) and environment-specific taxonomic class weights so that a multinomial naive Bayes classifier (scikit-learn MultinomialNB via q2-feature-classifier) can be trained once with bespoke EMPO 3 habitat weights and once with uniform weights for controlled comparison.

## When to use

Use when building taxonomic classifiers for amplicon ASV/OTU queries where species-level accuracy depends on reference class priors — e.g., importing Greengenes 99% reference data into QIIME 2 and assembling EMPO 3 habitat-specific or uniform class_weight inputs for fit-classifier-naive-bayes.

## When NOT to use

- Ready-made pretrained classifiers for the sample habitat (e.g., animal-distal-gut, soil-non-saline, human-stool) already exist on Zenodo — download them instead of retraining
- The query data are metagenomic profiles (MetaPhlAn2-style) rather than 16S marker-gene amplicon sequences, since clawback weights are built from marker-gene sample distributions
- The sample environment lacks sufficient characterization to assemble bespoke weights and no matching readytowear weights exist — use global average weights instead of forcing habitat-specific preparation

## Inputs

- Greengenes 99% OTU representative reference sequences (FASTA)
- Greengenes 99% OTU taxonomy (ranked lineage strings)
- EMPO 3 habitat-specific taxonomic class weights from readytowear (FeatureData[ClassWeight])
- Uniform taxonomic class weights (FeatureData[ClassWeight] baseline)
- Held-out query amplicon sequence variants per EMPO 3 habitat

## Outputs

- QIIME 2 reference read and taxonomy artifacts (.qza)
- Trained multinomial naive Bayes classifiers (.qza), one bespoke and one uniform per habitat
- Species-level classifications of query ASVs at confidence 0.7
- fig1_error_rates.csv with per-habitat and overall average error rates for both weighting schemes

## How to apply

Download the Greengenes 99% OTU reference sequences and taxonomy and import them into QIIME 2 as FeatureData[Sequence] and FeatureData[Taxonomy] artifacts for the q2-feature-classifier plugin. Fetch ready-made EMPO 3 habitat-specific taxonomic class weights from the readytowear repository (BenKaehler/readytowear), or assemble weights from scratch with q2-clawback by fetching Qiita samples (fetch_Qiita_samples, assemble_weights_from_Qiita, generate_class_weights) and computing taxonomic abundance distributions for the target habitat. Prepare a uniform-weight baseline as a matched FeatureData[ClassWeight] artifact. Train the naive Bayes classifier per habitat with q2-feature-classifier fit-classifier-naive-bayes using class_weight='balanced' bespoke weights and again with uniform weights, then verify both classifiers accept the same reference reads/taxonomy pair before running classify-sklearn at confidence 0.7.

## Related tools

- **QIIME 2** (framework hosting reference data artifacts and the feature-classifier plugin used to import and train classifiers) — https://qiime2.org
- **q2-feature-classifier** (fit-classifier-naive-bayes trains the MultinomialNB classifier on Greengenes 99% references with class_weight='balanced'; classify-sklearn assigns taxonomy at confidence 0.7)
- **q2-clawback** (assembles environment-specific taxonomic class weights from Qiita samples when ready-made weights are not used) — https://github.com/BenKaehler/q2-clawback
- **readytowear** (supplies ready-made EMPO 3 habitat taxonomic weights so clawback itself need not be run) — https://github.com/BenKaehler/readytowear
- **redbiom** (rapid sample discovery and feature characterization used to find and fetch Qiita samples underlying the weights) — https://github.com/biocore/redbiom
- **scikit-learn** (provides the MultinomialNB estimator underlying fit-classifier-naive-bayes)

## Examples

```
qiime feature-classifier fit-classifier-naive-bayes --i-reference-reads gg99-seqs.qza --m-reference-taxonomy gg99-tax.qza --p-class-weight empo3-weights.qza --p-class-weight--balanced --o-classifier bespoke-classifier.qza
```

## Evaluation signals

- Bespoke classifiers reproduce the reported overall average species-level error rate near 14% versus 25% for uniform weights across the 14 EMPO 3 habitats
- Paired t-test on per-habitat error rates yields P ≈ 5.8 × 10−5 in favor of bespoke weights
- Each trained classifier artifact validates against the same Greengenes 99% reference reads and taxonomy pair (same feature IDs)
- Bespoke species-level error is favourably comparable to existing classifiers' genus-level error rate of 16%
- fig1_error_rates.csv contains paired per-habitat error rates for all 14 EMPO 3 habitats under both weighting schemes

## Limitations

- Classification accuracy of naive Bayes classifiers degrades linearly with the degree to which the equal-abundance reference assumption is violated, and in practice it is always violated
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation), and empirical species distributions propagate misannotated sample types or sequencing biases into weights
- Performance degradation was observed for rare species under bespoke weighting
- Natural sample types lacking sufficient characterization cannot have bespoke weights assembled; average weights from global microbial species distributions are then superior to uniform weights
