---
name: random-forest-classifier-training
description: Use when a Deblur BIOM feature table with sample metadata (e.g. EMPO
  level 2/3 labels) is available and the goal is supervised, cross-validated classification
  of samples into environment categories from abundance data alone.
when_to_use_negative:
- The data are shotgun metagenomic, 18S/ITS, or metabolomic samples (e.g. EMP500 multi-omics)
  rather than the 16S Deblur tag-sequence table
- The goal is source tracking / microbial source attribution (e.g. SourceTracker2
  Gibbs sampling) rather than categorical environment classification
- Samples lack EMPO metadata annotations, so no labeled training set for EMPO level
  2/3 can be assembled
edam_operation: http://edamontology.org/operation_0370
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: randomForest
  role: Random forest classifier used as the 'rf' method to fit EMPO label models
- name: caret
  role: Orchestrates repeated 5x10-fold cross-validation and confusion-matrix computation
    over the random forest models
- name: R
  role: Language used for the meta-analysis notebooks and scripts that train and evaluate
    the random forest classifiers
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Deblur
  role: Exact-sequence de novo OTU picking method that produced the 90-bp V4 tag-sequence
    feature table used as classifier input
  repo: https://github.com/cuttlefishh/deblur
- name: Pandas
  role: Handles the sample metadata table (data type, variable type, sample counts
    per field) during subsetting to EMPO-annotated samples
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:292021e9fcc086e4c37b074357332a69538277c3786b125686ce9c8ee2393433
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Pandas
  - Python
  - R
  - caret
  - randomForest
  edam_operation: http://edamontology.org/operation_0370
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# random-forest-classifier-training

## Summary

Trains random forest classifiers on rarefied 16S feature tables to predict sample provenance categories (EMPO level 2 and level 3) from microbial community composition, replicating the EMP classification reported in the Earth Microbiome Project catalogue.

## When to use

Use when a Deblur BIOM feature table with sample metadata (e.g. EMPO level 2/3 labels) is available and the goal is supervised, cross-validated classification of samples into environment categories from abundance data alone.

## When NOT to use

- The data are shotgun metagenomic, 18S/ITS, or metabolomic samples (e.g. EMP500 multi-omics) rather than the 16S Deblur tag-sequence table
- The goal is source tracking / microbial source attribution (e.g. SourceTracker2 Gibbs sampling) rather than categorical environment classification
- Samples lack EMPO metadata annotations, so no labeled training set for EMPO level 2/3 can be assembled

## Inputs

- Deblur BIOM feature table (90-bp V4 16S tag sequences) from the EMP 16S Release 1 catalogue (~25,000 samples, 97 studies)
- Quality-filtered mapping/metadata file with EMPO level 2 and EMPO level 3 annotations
- Subset of 2,000 samples with EMPO level 2/3 metadata

## Outputs

- rf_accuracy_table.csv with confusion matrices and accuracy metrics for EMPO level 2 (~91%) and level 3 (~84%)
- Trained randomForest models (one per EMPO level) with out-of-fold predictions

## How to apply

Load the Deblur BIOM feature table and quality-filtered mapping file for the ~25,000-sample EMP 16S Release 1 catalogue, then subset to the 2,000 samples carrying EMPO level 2/3 annotations. Rarefy the feature table to 5,000 observations per sample and exclude sequences annotated as chloroplast before computing statistics. Train a random forest classifier with the R randomForest package via caret, using repeated 5x10-fold cross-validation on EMPO level 2 labels (animal-associated, plant-associated, saline, non-saline free-living), and a second model with the same scheme on the fine-scale EMPO level 3 environment labels. Rationale: rarefaction to a fixed depth (5,000) removes uneven sequencing effort as a confounder, and chloroplast sequences are non-microbial contaminants that would leak sample provenance. Compute out-of-fold confusion matrices and overall accuracy for both levels; acceptance is met when cross-validated accuracies match the reported 91% (EMPO level 2) and 84% (EMPO level 3).

## Related tools

- **randomForest** (Random forest classifier used as the 'rf' method to fit EMPO label models)
- **caret** (Orchestrates repeated 5x10-fold cross-validation and confusion-matrix computation over the random forest models)
- **R** (Language used for the meta-analysis notebooks and scripts that train and evaluate the random forest classifiers) — https://github.com/biocore/emp/tree/master/methods
- **Deblur** (Exact-sequence de novo OTU picking method that produced the 90-bp V4 tag-sequence feature table used as classifier input) — https://github.com/cuttlefishh/deblur
- **Pandas** (Handles the sample metadata table (data type, variable type, sample counts per field) during subsetting to EMPO-annotated samples)

## Examples

```
model <- caret::train(x = X, y = empo2, method = "rf", trControl = trainControl(method = "repeatedcv", number = 10, repeats = 5))
```

## Evaluation signals

- Cross-validated overall accuracy reaches ~91% for EMPO level 2 (animal-associated, plant-associated, saline, non-saline free-living) and ~84% for EMPO level 3
- Out-of-fold confusion matrices are computed for both classification levels and written to rf_accuracy_table.csv
- Feature table was rarefied to exactly 5,000 observations per sample before model training
- Sequences annotated as chloroplast were removed prior to computing any statistics
- Cross-validation used a repeated 5x10-fold scheme (5 repeats of 10-fold CV) on the 2,000-sample Deblur subset

## Limitations

- Accuracies are tied to the 2,000-sample subset with EMPO level 2/3 annotations and the 5,000-read rarefaction depth; other depths or subsets may not reproduce the 91%/84% figures
- Only 16S amplicon data processed with Deblur (exact sequences, not clustered OTUs) support cross-study comparability at the level reported
- Samples without the rarefaction depth of 5,000 observations must be dropped, which can bias the training set if shallow samples dominate some environments

## Evidence

- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Chloroplast exclusion: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] EMP 16S Release 1 contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [supplementary] Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm.: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [abstract] Deblur rationale: "Coordinated protocols and new analytical methods, particularly the use of exact sequences instead of clustered operational taxonomic units, enable bacterial and archaeal ribosomal RNA gene sequences"
