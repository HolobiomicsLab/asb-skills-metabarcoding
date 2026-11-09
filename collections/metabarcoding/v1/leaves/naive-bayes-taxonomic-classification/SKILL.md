---
name: naive-bayes-taxonomic-classification
description: Use when 16S/amplicon sequence variants need taxonomy assigned at species
  level with QIIME 2, especially when environment-specific class weights (e.g., EMPO
  3 habitat weights) can replace the untenable uniform-weights assumption.
when_to_use_negative:
- Input data are not marker-gene amplicon sequences against a reference taxonomy (e.g.,
  shotgun metagenomic profiles better served by MetaPhlAn2)
- The reference database is poorly curated (misannotated sequences) — bespoke classification
  is not immune to reference misannotation errors
- A taxonomic assignment already exists for the feature table and retraining would
  only duplicate prior work
edam_operation: http://edamontology.org/operation_0392
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: q2-feature-classifier
  role: Trains and runs the scikit-learn multinomial naive Bayes taxonomy classifier
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: QIIME 2
  role: Framework hosting the classifier plugin and artifact types for references,
    reads, and taxonomies
  repo: https://qiime2.org
- name: q2-clawback
  role: Assembles environment-specific class weights that re-weight the naive Bayes
    classifier
  repo: https://github.com/BenKaehler/q2-clawback
- name: scikit-learn
  role: Provides the multinomial naive Bayes implementation and regression used to
    model error-rate trends
  repo: https://scikit-learn.org
- name: imbalanced-learn
  role: Conventions used to enable class weights in the classifier, countering reference
    class imbalance
  repo: https://github.com/scikit-learn-contrib/imbalanced-learn
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:2a560d11207e70ff35aa9a3da9f170cb749154d5dfd2dc07521ba5ab1ac7e699
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
  edam_operation: http://edamontology.org/operation_0392
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# naive-bayes-taxonomic-classification

## Summary

Species-level taxonomic classification of marker-gene amplicon sequences using a scikit-learn multinomial naive Bayes classifier trained via QIIME 2's q2-feature-classifier, optionally re-weighted with environment-specific (bespoke) class weights from q2-clawback to reduce species-level error rates from 25% to 14%.

## When to use

Use when 16S/amplicon sequence variants need taxonomy assigned at species level with QIIME 2, especially when environment-specific class weights (e.g., EMPO 3 habitat weights) can replace the untenable uniform-weights assumption.

## When NOT to use

- Input data are not marker-gene amplicon sequences against a reference taxonomy (e.g., shotgun metagenomic profiles better served by MetaPhlAn2)
- The reference database is poorly curated (misannotated sequences) — bespoke classification is not immune to reference misannotation errors
- A taxonomic assignment already exists for the feature table and retraining would only duplicate prior work

## Inputs

- Reference sequence reads and full-rank taxonomy (e.g., Greengenes) as QIIME 2 artifacts for classifier training
- Amplicon sequence variants (e.g., Deblur/DADA2 features from 16S V4 reads)
- Optional taxonomic class weights (FeatureWeights) assembled by q2-clawback from Qiita/redbiom sample abundances

## Outputs

- Trained naive Bayes classifier artifact (q2-feature-classifier .qza)
- Per-sequence taxonomy classifications at species level (confidence = -1)
- Species-level classification error rates per comparison (e.g., written to figures_3_4_results.csv)

## How to apply

Train the classifier with q2-feature-classifier by fitting a scikit-learn multinomial naive Bayes model on reference reads (e.g., Greengenes) and their taxonomy, with class weights enabled via imbalanced-learn conventions so abundant taxa are favored over the uniform prior. To force assignments to the species level, run classification with confidence set to -1 (no rejection). For best accuracy, supply class weights assembled by q2-clawback from source-environment samples (bespoke weights for 14 EMPO 3 habitats across 21,513 samples, or ready-made weights from readytowear) rather than uniform weights, since accuracy degrades linearly as the uniform assumption is violated. Evaluate by computing species-level error rates, expecting an overall decline from 25% (uniform) to 14% (bespoke), comparable to other classifiers' genus-level error rate of 16%.

## Related tools

- **q2-feature-classifier** (Trains and runs the scikit-learn multinomial naive Bayes taxonomy classifier) — https://github.com/qiime2/q2-feature-classifier/
- **QIIME 2** (Framework hosting the classifier plugin and artifact types for references, reads, and taxonomies) — https://qiime2.org
- **q2-clawback** (Assembles environment-specific class weights that re-weight the naive Bayes classifier) — https://github.com/BenKaehler/q2-clawback
- **scikit-learn** (Provides the multinomial naive Bayes implementation and regression used to model error-rate trends) — https://scikit-learn.org
- **imbalanced-learn** (Conventions used to enable class weights in the classifier, countering reference class imbalance) — https://github.com/scikit-learn-contrib/imbalanced-learn

## Examples

```
redbiom search metadata beer | head | redbiom fetch samples --context $ctx --output beer_example.biom
```

## Evaluation signals

- Species-level average error rate with bespoke weights falls to ~14%, versus ~25% under uniform weights and 16% genus-level for existing classifiers
- Cross-habitat weight experiments reproduce 117/182 comparisons where cross-habitat weights beat uniform weights, with Pearson r2 = 0.57 (P < 2.2e-16) between error rate and weight divergence
- All classifications run with confidence = -1 return species-level (rank 7) assignments with no rejections for every fetched sample
- Classifier artifact is a scikit-learn multinomial naive Bayes model with class weights enabled via imbalanced-learn conventions
- Downstream generate_class_weights output (classified_abundances.csv) contains classified taxonomic abundances for every sample in the habitat

## Limitations

- Some degradation in performance was observed for rare species under bespoke weighting
- The uniform-weights assumption that all reference species are equally likely to be observed is untenable and imposes a consistently negative impact, even versus deliberately incorrect cross-habitat weights
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation) or sequencing biases propagated into the weights
- Natural sample types lacking sufficient characterization cannot have bespoke weights assembled; average weights estimated from global distributions are then superior to uniform weights

## Evidence

- [abstract] Uniform assumption untenable: "Popular naive Bayes taxonomic classifiers for amplicon sequences assume that all species in the reference database are equally likely to be observed."
- [abstract] Error-rate improvement: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [discussion] Rare-species degradation: "We did observe some degradation in performance for rare species"
- [readme] Ready-made weights usage: [evidence span withheld — anchor status: external_doc]
