---
name: repeated-cross-validation-model-evaluation
description: Use when a prediction task over high-dimensional feature tables (e.g.,
  Deblur BIOM counts vs. EMPO labels) needs a rigorous out-of-fold accuracy estimate
  rather than a single train/test split.
when_to_use_negative:
- The question is unsupervised structure discovery (e.g., beta-diversity ordination)
  with no labelled response variable
- The feature table has not been preprocessed to match the training distribution (e.g.,
  unrarefied counts or chloroplast sequences retained), making accuracy estimates
  incomparable to the reported benchmarks
- Sample size per class is too small for 10-fold stratification, so fold-wise label
  imbalance invalidates the accuracy comparison
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3697
tools:
- name: randomForest
  role: Random forest classifier trained on rarefied community composition to predict
    EMPO environment labels
- name: caret
  role: R framework implementing the repeated 5x10-fold cross-validation scheme and
    accuracy extraction
- name: Python
  role: Subsetting the 2,000-sample Deblur subset and loading/rarefying the BIOM table
    before R model training
- name: Pandas
  role: Manipulating the mapping file metadata (EMPO level 2/3 labels) paired with
    the feature table
- name: Deblur
  role: Source of the exact-sequence (90-bp V4) feature table used as classifier input
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:a4bed3c2241aceba154050e0e4a52ca8ba6edfaedb5b8c1be698ef65174b8232
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Pandas
  - Python
  - R
  - caret
  - randomForest
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# Repeated cross-validation model evaluation

## Summary

Evaluates supervised classifiers (e.g., random forest on EMP microbial community features) using repeated k-fold cross-validation and reports confusion-matrix-based accuracy against published reference values.

## When to use

Use when a prediction task over high-dimensional feature tables (e.g., Deblur BIOM counts vs. EMPO labels) needs a rigorous out-of-fold accuracy estimate rather than a single train/test split.

## When NOT to use

- The question is unsupervised structure discovery (e.g., beta-diversity ordination) with no labelled response variable
- The feature table has not been preprocessed to match the training distribution (e.g., unrarefied counts or chloroplast sequences retained), making accuracy estimates incomparable to the reported benchmarks
- Sample size per class is too small for 10-fold stratification, so fold-wise label imbalance invalidates the accuracy comparison

## Inputs

- Deblur BIOM feature table (90-bp V4 16S tag sequences from EMP 16S Release 1)
- Quality-filtered EMP mapping file with EMPO level 2/3 environment annotations for ~25,000 samples
- Cross-validation scheme specification (5 repeats x 10 folds)

## Outputs

- rf_accuracy_table.csv with confusion matrices and overall accuracy per classification level
- Out-of-fold predicted labels per sample
- Cross-validated accuracy metrics (target ~91% EMPO level 2, ~84% EMPO level 3)

## How to apply

Load a processed feature table (e.g., the Deblur BIOM table from EMP 16S Release 1) together with the quality-filtered mapping file, and subset samples carrying the metadata labels of interest (here EMPO level 2/3 annotations for ~2,000 samples). Apply the same sample filters used in the paper before statistics: rarefy to 5,000 observations per sample and exclude sequences annotated as chloroplast. Train the model (random forest via R caret/randomForest) with repeated 5x10-fold cross-validation on the coarse labels, then repeat on the fine-scale labels, extracting out-of-fold predictions at each fold. Compute overall accuracy and confusion matrices from the pooled out-of-fold predictions; acceptance is judged by agreement with reported accuracies (91% for EMPO level 2, 84% for EMPO level 3).

## Related tools

- **randomForest** (Random forest classifier trained on rarefied community composition to predict EMPO environment labels)
- **caret** (R framework implementing the repeated 5x10-fold cross-validation scheme and accuracy extraction)
- **Python** (Subsetting the 2,000-sample Deblur subset and loading/rarefying the BIOM table before R model training)
- **Pandas** (Manipulating the mapping file metadata (EMPO level 2/3 labels) paired with the feature table)
- **Deblur** (Source of the exact-sequence (90-bp V4) feature table used as classifier input) — https://github.com/cuttlefishh/deblur

## Examples

```
Rscript -e 'ctrl <- trainControl(method="repeatedcv", number=10, repeats=5); m <- train(empo2 ~ ., data=rarified_tbl, method="rf", trControl=ctrl); print(confusionMatrix(m))'
```

## Evaluation signals

- Cross-validated accuracy on EMPO level 2 labels reaches ~91%, matching the reported value
- Cross-validated accuracy on EMPO level 3 environment labels reaches ~84%
- Confusion matrices are computed from out-of-fold predictions only (no training-fold leakage)
- Feature table was rarefied to 5,000 observations per sample and chloroplast sequences excluded before any statistics
- rf_accuracy_table.csv exists with both confusion matrices and accuracy rows

## Limitations

- Accuracy is tied to the 2,000-sample subset with EMPO level 2/3 annotations; extending to other sample subsets or label schemes invalidates direct comparison
- Rarefaction to 5,000 observations discards samples below that depth and changes feature abundances, so results do not transfer to unrarefied tables
- Performance depends on the exact Deblur exact-sequence features; substituting clustered OTUs or different reference processing is not equivalent

## Evidence

- [supplementary] Rarefaction and chloroplast filtering precede statistics: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Chloroplast sequences excluded before statistics: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] Source of the input feature table and mapping files: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [supplementary] Feature sequences come from Deblur V4 amplicons: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
