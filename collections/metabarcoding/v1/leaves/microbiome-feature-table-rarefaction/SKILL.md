---
name: microbiome-feature-table-rarefaction
description: Use when a feature table (Deblur or OTU BIOM) with uneven per-sample
  sequence counts feeds into cross-sample statistics (e.g., random forest classification
  of EMPO environment labels), and per-sample depths must be equalized to 5,000 observations
  first.
when_to_use_negative:
- The goal is prevalence/abundance feature filtering (e.g., mc2 — minimum sequence
  count in an OTU of 2) rather than per-sample depth normalization; these are distinct
  filtering steps.
- The input is a closed-reference OTU table against a different reference (Greengenes
  13.8 / Silva 123) at a different processing context — the 5,000-observation depth
  was specified for the Deblur tag-sequences table.
- The analysis requires total unfiltered abundance per sample (e.g., exact read-count
  reporting) rather than depth-normalized comparisons.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0092
tools:
- name: Python
  role: Loads and subsets the Deblur BIOM feature table and quality-filtered mapping
    file prior to rarefaction
- name: Pandas
  role: Handles metadata fields and sample subsetting for the 27,751-sample metadata
    table
- name: Deblur
  role: Produces the exact-sequence (90-bp V4) feature table that is rarefied to 5,000
    observations per sample
  repo: https://github.com/cuttlefishh/deblur
- name: R
  role: Hosts the downstream random forest analysis (randomForest, caret) applied
    to the rarefied table
- name: redbiom
  role: Retrieves sample data from the EMP catalogue into BIOM format for a chosen
    processing context before rarefaction
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:eff15f6df654e18e5691417e0fa2db739887678e7eb76459d389be7139899ffa
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
  - redbiom
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# microbiome-feature-table-rarefaction

## Summary

Normalizes 16S amplicon feature tables (e.g., the Deblur BIOM tables from EMP 16S Release 1) to equal sequencing depth — 5,000 observations per sample — before downstream statistics, and excludes contaminant lineages such as chloroplast sequences. This is the depth-standardization step used in the EMP meta-analysis to make cross-study community comparisons and machine-learning classification comparable.

## When to use

Use when a feature table (Deblur or OTU BIOM) with uneven per-sample sequence counts feeds into cross-sample statistics (e.g., random forest classification of EMPO environment labels), and per-sample depths must be equalized to 5,000 observations first.

## When NOT to use

- The goal is prevalence/abundance feature filtering (e.g., mc2 — minimum sequence count in an OTU of 2) rather than per-sample depth normalization; these are distinct filtering steps.
- The input is a closed-reference OTU table against a different reference (Greengenes 13.8 / Silva 123) at a different processing context — the 5,000-observation depth was specified for the Deblur tag-sequences table.
- The analysis requires total unfiltered abundance per sample (e.g., exact read-count reporting) rather than depth-normalized comparisons.

## Inputs

- Deblur BIOM feature table (90-bp V4 tag sequences) from EMP 16S Release 1 (~25,000 samples, 97 studies)
- Quality-filtered mapping file with EMPO level 2/3 environment metadata
- Sequence annotations used to identify chloroplast-assigned sequences

## Outputs

- Rarefied BIOM feature table with 5,000 observations per sample
- Feature table excluding chloroplast-annotated sequences, ready for statistics
- Downstream rf_accuracy_table.csv with confusion matrices and cross-validated accuracy (~91% EMPO level 2, ~84% EMPO level 3)

## How to apply

Load the merged, quality-filtered mapping file and the Deblur BIOM feature table for the ~25,000-sample EMP 16S Release 1 catalogue, then subset to the samples with EMPO level 2/3 annotations (~2,000 samples). Rarefy the feature table to a fixed depth of 5,000 observations per sample — the same threshold used across the EMP supplementary analyses — dropping samples with fewer than 5,000 reads. Before computing any statistics, exclude sequences annotated as chloroplast from the rarefied table. The rarefied, chloroplast-filtered table is then the input for downstream steps such as random forest classifiers (R randomForest/caret, repeated 5x10-fold cross-validation) that distinguish animal-associated, plant-associated, saline, and non-saline free-living samples. Rationale: equal depth removes sequencing-depth confounding from feature-abundance vectors, and removing chloroplast sequences avoids plant-host signal being counted as microbial community.

## Related tools

- **Python** (Loads and subsets the Deblur BIOM feature table and quality-filtered mapping file prior to rarefaction)
- **Pandas** (Handles metadata fields and sample subsetting for the 27,751-sample metadata table)
- **Deblur** (Produces the exact-sequence (90-bp V4) feature table that is rarefied to 5,000 observations per sample) — https://github.com/cuttlefishh/deblur
- **R** (Hosts the downstream random forest analysis (randomForest, caret) applied to the rarefied table)
- **redbiom** (Retrieves sample data from the EMP catalogue into BIOM format for a chosen processing context before rarefaction) — https://github.com/biocore/redbiom

## Examples

```
ctrl <- trainControl(method="repeatedcv", number=10, repeats=5); rf_fit <- train(t(rarefied_table), empo2_labels, method="rf", trControl=ctrl)
```

## Evaluation signals

- Every retained sample in the rarefied table has exactly 5,000 observations; samples below 5,000 reads are dropped.
- No sequences annotated as chloroplast remain in the table used for statistics.
- Subsequent random forest cross-validation (repeated 5x10-fold) reproduces ~91% accuracy on EMPO level 2 and ~84% on EMPO level 3 labels.
- Confusion matrices and accuracy metrics are written to rf_accuracy_table.csv as specified in the workflow.
- Sample IDs in the rarefied table match the subset of samples carrying EMPO level 2/3 annotations in the quality-filtered mapping file.

## Limitations

- Samples with fewer than 5,000 observations cannot participate in the rarefied analysis, reducing the effective sample set.
- Rarefaction discards information in deeper samples (subsample of counts), which affects abundance-sensitive statistics.
- The 5,000-observation depth and chloroplast exclusion are specific to the EMP 16S Release 1 Deblur processing; reusing them for other processing contexts (closed-reference OTU picking against Greengenes 13.8 or Silva 123) may not be appropriate.

## Evidence

- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Chloroplast exclusion before statistics: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] EMP 16S Release 1 contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [readme] redbiom sample retrieval: [evidence span withheld — anchor status: external_doc]
