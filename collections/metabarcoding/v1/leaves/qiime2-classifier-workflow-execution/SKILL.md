---
name: qiime2-classifier-workflow-execution
description: Use when a marker-gene amplicon dataset must be taxonomically classified
  in QIIME 2, and the goal is to benchmark species-level classification with environment-specific
  (bespoke) class weights against genus-level uniform-weight classification.
when_to_use_negative:
- The query sample's environment lacks sufficient characterization to assemble bespoke
  weights (e.g., a poorly characterized natural sample type), in which case average
  weights estimated from global microbial species distributions or uniform weights
  are the appropriate alternative.
- The data are not marker-gene amplicon sequences classifiable against a QIIME 2 reference
  database (e.g., whole-genome metagenomes handled by MetaPhlAn2), or no ground-truth
  taxonomy is available to compute error rates.
- Highly unusual sample distributions (e.g., synthetic populations) where ready-made
  weights do not apply — custom bespoke weights must first be compiled from existing
  samples.
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3168
tools:
- name: q2-feature-classifier
  role: Runs the classify-sklearn naive Bayes classification under bespoke vs. uniform
    class weights against the Greengenes 13_8 99% OTU reference
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: q2-clawback
  role: Assembles environment-specific (Qiita-derived) taxonomic class weights; ready-made
    weights distributed via readytowear
  repo: https://github.com/BenKaehler/q2-clawback
- name: readytowear
  role: Source of ready-made taxonomic weights for common primer, database, and sample
    habitat (EMPO 3) combinations
  repo: https://github.com/BenKaehler/readytowear
- name: redbiom
  role: Fetches sample data and metadata from Qiita by metadata search or feature/taxon
    search into BIOM for weight assembly and test-set construction
  repo: https://github.com/biocore/redbiom
- name: QIIME 2
  role: Hosts the classification workflow plugins and Bray-Curtis metric computation
  repo: https://qiime2.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:fcf2cfb573033fc39045df8f2ae487c3e03f7052ed8635a325e38c0bfb2daaed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qiime2/q2-feature-classifier
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - readytowear
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# qiime2-classifier-workflow-execution

## Summary

Executes the QIIME 2 taxonomic classification workflow of the Kaehler et al. (2019) Nature Communications paper: classifying marker-gene amplicon sequences with q2-feature-classifier's naive Bayes classifier under species-level bespoke (environment-specific) taxonomic weights versus genus-level uniform weights against the Greengenes 13_8 99% OTU reference, then quantifying error rates and Bray-Curtis dissimilarity.

## When to use

Use when a marker-gene amplicon dataset must be taxonomically classified in QIIME 2, and the goal is to benchmark species-level classification with environment-specific (bespoke) class weights against genus-level uniform-weight classification.

## When NOT to use

- The query sample's environment lacks sufficient characterization to assemble bespoke weights (e.g., a poorly characterized natural sample type), in which case average weights estimated from global microbial species distributions or uniform weights are the appropriate alternative.
- The data are not marker-gene amplicon sequences classifiable against a QIIME 2 reference database (e.g., whole-genome metagenomes handled by MetaPhlAn2), or no ground-truth taxonomy is available to compute error rates.
- Highly unusual sample distributions (e.g., synthetic populations) where ready-made weights do not apply — custom bespoke weights must first be compiled from existing samples.

## Inputs

- Amplicon sequence variants / test amplicon sequences (e.g., Deblur- or DADA2-derived, or fetched via redbiom)
- Pre-fitted Greengenes-trained naive Bayes classifier artifact from q2-feature-classifier
- Bespoke class-weight artifacts per EMPO 3 habitat type (from BenKaehler/readytowear or generated via q2-clawback)
- Greengenes 13_8 99% OTU reference taxonomy/sequences
- Ground-truth taxonomies for test samples (for error-rate computation)

## Outputs

- Per-sample species-level (bespoke) and genus-level (uniform) error rates
- Per-sample Bray-Curtis dissimilarities between predicted and true community compositions
- Single-sided paired t-test t-statistics and P-values
- Summary CSV (e.g., fig1_comparison.csv) of mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values

## How to apply

Load the pre-fitted Greengenes-trained naive Bayes classifier from q2-feature-classifier and pull bespoke class weights for the test environment (one of the 14 EMPO 3 habitat types) from the readytowear repository (https://github.com/BenKaehler/readytowear). Run two classification passes with q2-feature-classifier's classify-sklearn against the Greengenes 13_8 99% OTU reference: species-level classification using the bespoke (Qiita-derived) weights, and genus-level classification using uniform weights. Compare predicted taxonomies against known ground-truth taxonomies to compute per-sample error rates at species level (bespoke) and genus level (uniform), and compute Bray-Curtis dissimilarity between predicted and true community compositions per condition. Run a single-sided paired t-test comparing matched samples for both error rate and Bray-Curtis dissimilarity, then compile mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values (expect ~14% vs ~16% error, ~0.13 vs ~0.15 Bray-Curtis, P = 0.013). The rationale is that naive Bayes classifiers assume all reference species are equally likely to be observed, and this assumption is 'always violated' in practice, degrading accuracy linearly.

## Related tools

- **q2-feature-classifier** (Runs the classify-sklearn naive Bayes classification under bespoke vs. uniform class weights against the Greengenes 13_8 99% OTU reference) — https://github.com/qiime2/q2-feature-classifier/
- **q2-clawback** (Assembles environment-specific (Qiita-derived) taxonomic class weights; ready-made weights distributed via readytowear) — https://github.com/BenKaehler/q2-clawback
- **readytowear** (Source of ready-made taxonomic weights for common primer, database, and sample habitat (EMPO 3) combinations) — https://github.com/BenKaehler/readytowear
- **redbiom** (Fetches sample data and metadata from Qiita by metadata search or feature/taxon search into BIOM for weight assembly and test-set construction) — https://github.com/biocore/redbiom
- **QIIME 2** (Hosts the classification workflow plugins and Bray-Curtis metric computation) — https://qiime2.org

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier gg-99-naive-bayes-classifier.qza --i-reads test-reads.qza --i-class-weight bespoke-weights.qza --o-classification species-preds.qza
```

## Evaluation signals

- Mean species-level bespoke error rate is approximately 14% and genus-level uniform error rate approximately 16% across the 14 EMPO 3 habitat types, matching Fig. 1 of the paper.
- Mean Bray-Curtis dissimilarity is approximately 0.13 (bespoke species) versus 0.15 (uniform genus).
- Single-sided paired t-test on matched samples yields P = 0.013 for the species-vs-genus comparison.
- Output CSV (fig1_comparison.csv) contains mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values for both classification conditions.
- Classification must be against the Greengenes 13_8 99% OTU reference using the q2-feature-classifier naive Bayes classifier, with weights restricted to readytowear EMPO 3 habitat types (e.g., animal-distal-gut, soil-non-saline, human-stool).

## Limitations

- Performance degradation was observed for rare species under bespoke weighting.
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation).
- Empirical species distributions introduce potential error sources such as misannotated sample types and sequencing biases propagated into taxonomic weights; systematic sequencing bias would require integrating multiple technologies to control.

## Evidence

- [abstract] At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%).: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [abstract] By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types.: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [methods] please check out our ready-made weights at [readytowear](https://github.com/BenKaehler/readytowear): "please check out our ready-made weights at [readytowear](https://github.com/BenKaehler/readytowear)"
- [readme] Ready-made taxonomic weights generated by q2-clawback for use with the q2-feature-classifier taxonomy classifier.: [evidence span withheld — anchor status: external_doc]
- [readme] pretrained classifiers are available for download from [Zenodo](https://zenodo.org/record/6395539): [evidence span withheld — anchor status: external_doc]
