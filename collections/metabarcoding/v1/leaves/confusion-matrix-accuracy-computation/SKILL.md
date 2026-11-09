---
name: confusion-matrix-accuracy-computation
description: Use when classifying EMP samples into EMPO level 2/3 environment categories
  from Deblur community composition and you must report cross-validated confusion
  matrices with overall accuracy (target ~91% and ~84%).
when_to_use_negative:
- Input is raw amplicon reads rather than a Deblur-processed BIOM feature table —
  the sequences must first be processed with Deblur into exact-sequence features.
- Samples lack EMPO level 2/3 environment annotations, so no reliable class labels
  exist for supervised training or a meaningful confusion matrix.
- The goal is feature/taxon-specific sample search rather than supervised classification
  — redbiom metadata and feature searches are the appropriate tool instead.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: randomForest/caret (R)
  role: Train random forest classifiers with repeated 5x10-fold cross-validation on
    EMPO labels and derive confusion matrices and accuracy
- name: Deblur
  role: Produced the exact-sequence 90-bp V4 tag feature table that is the classification
    input
  repo: https://github.com/cuttlefishh/deblur
- name: redbiom
  role: Retrieval of sample data and metadata from the EMP catalogue into BIOM and
    mapping files for subsetting
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:15a1b13feb9bf71d62af8b6f3751cf5a3d1cc82142af30e7d0a4190b951afe7d
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
  - randomForest/caret (R)
  - redbiom
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# confusion-matrix-accuracy-computation

## Summary

Builds out-of-fold confusion matrices and overall accuracy metrics for supervised classification of EMP 16S samples into EMPO environment categories, serving as the validation step for random-forest provenance classification.

## When to use

Use when classifying EMP samples into EMPO level 2/3 environment categories from Deblur community composition and you must report cross-validated confusion matrices with overall accuracy (target ~91% and ~84%).

## When NOT to use

- Input is raw amplicon reads rather than a Deblur-processed BIOM feature table — the sequences must first be processed with Deblur into exact-sequence features.
- Samples lack EMPO level 2/3 environment annotations, so no reliable class labels exist for supervised training or a meaningful confusion matrix.
- The goal is feature/taxon-specific sample search rather than supervised classification — redbiom metadata and feature searches are the appropriate tool instead.

## Inputs

- Deblur BIOM feature table (90-bp V4 tag sequences from the EMP 16S Release 1 catalogue)
- Quality-filtered mapping file with EMPO level 2/3 metadata annotations for ~25,000 samples
- R environment with randomForest and caret packages

## Outputs

- rf_accuracy_table.csv containing confusion matrices and overall accuracy metrics per EMPO classification level
- Per-level accuracy estimates (~91% for EMPO level 2, ~84% for EMPO level 3)

## How to apply

Load the Deblur BIOM feature table and quality-filtered mapping file for the ~25,000-sample EMP 16S Release 1 catalogue, and subset to the 2,000 samples carrying EMPO level 2/3 annotations. Rarefy the feature table to 5,000 observations per sample and exclude sequences annotated as chloroplast before statistics are computed. Train a random forest with the R caret/randomForest packages using repeated 5x10-fold cross-validation, once on EMPO level 2 labels (animal-associated, plant-associated, saline, non-saline free-living) and once on fine-scale EMPO level 3 environment labels. From the out-of-fold predictions, compute a confusion matrix and overall accuracy at each level; acceptance is met when reproduced accuracies match the reported 91% (EMPO level 2) and 84% (level 3), written to rf_accuracy_table.csv.

## Related tools

- **randomForest/caret (R)** (Train random forest classifiers with repeated 5x10-fold cross-validation on EMPO labels and derive confusion matrices and accuracy)
- **Deblur** (Produced the exact-sequence 90-bp V4 tag feature table that is the classification input) — https://github.com/cuttlefishh/deblur
- **redbiom** (Retrieval of sample data and metadata from the EMP catalogue into BIOM and mapping files for subsetting) — https://github.com/biocore/redbiom

## Examples

```
rf_fit <- train(x = t(feat_table), y = empo2_labels, method = "rf", trControl = trainControl(method = "repeatedcv", number = 10, repeats = 5))
```

## Evaluation signals

- Overall accuracy on EMPO level 2 labels reproduces the reported ~91% after repeated 5x10-fold cross-validation.
- Overall accuracy on fine-scale EMPO level 3 labels reproduces the reported ~84%.
- Confusion matrices are computed from out-of-fold predictions only — no training-fold leakage inflates accuracy.
- All samples entering the classifier are rarefied to exactly 5,000 observations per sample.
- Sequences annotated as chloroplast are absent from the feature table before statistics are computed.

## Limitations

- Only the 2,000 of ~25,000 EMP 16S Release 1 samples with EMPO level 2/3 annotations can be used for labeled classification; accuracy estimates are conditional on that subset.
- Accuracy depends on the rarefaction depth (5,000 observations/sample) and the exclusion of chloroplast-annotated sequences; results may shift with different filter choices.
- Classification is based on 90-bp V4 Deblur tag sequences (16S only), so performance does not transfer directly to other marker genes or sequencing depths.

## Evidence

- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Sequences annotated as chloroplast were excluded before statistics were computed.: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [readme] Redbiom is a cache service for sample metadata and sample data.: [evidence span withheld — anchor status: external_doc]
