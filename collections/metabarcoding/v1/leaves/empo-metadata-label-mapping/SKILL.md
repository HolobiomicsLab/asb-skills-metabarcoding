---
name: empo-metadata-label-mapping
description: Use when EMP 16S Release 1 samples need environment-category labels (EMPO
  level 2 or level 3) derived from the quality-filtered mapping file, e.g. to train
  or evaluate a classifier on community composition.
when_to_use_negative:
- The mapping file already contains the target environment labels and you only need
  community data (skip straight to feature-table rarefaction)
- You are working with samples outside EMP 16S Release 1 that lack EMPO annotations
  — labels cannot be derived from metadata that does not carry EMPO terms
- You need taxonomy-based label assignment (e.g. Greengenes OTU taxonomy) rather than
  sample environment metadata
edam_operation: http://edamontology.org/operation_0224
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: Python
  role: Used with Pandas to curate and subset the mapping-file metadata fields used
    for EMPO label extraction
- name: Pandas
  role: Handles the mapping-file variable types and per-field sample counts during
    EMPO label mapping
- name: redbiom
  role: Command-line tool to query the Qiita database, including EMP studies, to retrieve
    sample metadata for labelling
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:59ad3fecebb881215ef067832e21fecb7886c42415a7a6c37ba503a31ebe7613
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - Pandas
  - Python
  - R
  - caret
  - redbiom
  edam_operation: http://edamontology.org/operation_0224
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# empo-metadata-label-mapping

## Summary

Maps EMP sample metadata to EMPO (Earth Microbiome Project Ontology) environment labels at coarse (level 2: animal-associated, plant-associated, saline, non-saline free-living) and fine (level 3) scales, providing the class labels needed for environment classification from community composition.

## When to use

Use when EMP 16S Release 1 samples need environment-category labels (EMPO level 2 or level 3) derived from the quality-filtered mapping file, e.g. to train or evaluate a classifier on community composition.

## When NOT to use

- The mapping file already contains the target environment labels and you only need community data (skip straight to feature-table rarefaction)
- You are working with samples outside EMP 16S Release 1 that lack EMPO annotations — labels cannot be derived from metadata that does not carry EMPO terms
- You need taxonomy-based label assignment (e.g. Greengenes OTU taxonomy) rather than sample environment metadata

## Inputs

- EMP 16S Release 1 merged and quality-filtered mapping file (metadata for ~25,000 samples in 97 studies)
- EMPO level 2 and level 3 annotation columns within the mapping file
- Deblur BIOM feature table sample IDs for alignment

## Outputs

- Sample-to-EMPO level 2 label vector (animal-associated, plant-associated, saline free-living, non-saline free-living)
- Sample-to-EMPO level 3 fine-scale environment label vector
- Subset of ~2,000 samples with EMPO annotations ready for classifier training

## How to apply

Load the merged and quality-filtered mapping file accompanying EMP 16S Release 1 (~25,000 samples in 97 studies) and subset to the ~2,000 samples carrying EMPO level 2/3 annotations. Read the EMPO category columns directly from the mapping file metadata, which was curated with Python/Pandas as documented in the supplementary metadata table. Use the level 2 labels (animal-associated, plant-associated, saline free-living, non-saline free-living) as the coarse class vector and the level 3 environment labels as the fine-grained class vector, keeping sample IDs aligned with the Deblur BIOM feature table rows. These label vectors then serve as the response variables (y) for random forest cross-validation; label quality and one-to-one sample alignment with the rarefied feature table are what determine downstream classifier validity.

## Related tools

- **Python** (Used with Pandas to curate and subset the mapping-file metadata fields used for EMPO label extraction)
- **Pandas** (Handles the mapping-file variable types and per-field sample counts during EMPO label mapping)
- **redbiom** (Command-line tool to query the Qiita database, including EMP studies, to retrieve sample metadata for labelling) — https://github.com/biocore/redbiom

## Examples

```
redbiom fetch sample-metadata --context Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506 --output emp_mapping.txt
```

## Evaluation signals

- The subset of samples with EMPO annotations should be ~2,000 of the ~27,751 / ~25,000 catalogue samples
- Level 2 labels must resolve to exactly the four categories: animal-associated, plant-associated, saline free-living, non-saline free-living
- Every sample ID in the label vector must match a sample ID in the rarefied (5,000-observation) Deblur BIOM table
- Downstream cross-validated random forest accuracy should reach ~91% (EMPO level 2) and ~84% (level 3) if labels are mapped correctly

## Limitations

- Only samples with EMPO level 2/3 metadata annotations can be labelled; the wider ~25,000-sample catalogue lacks these annotations
- EMPO labels come from contributor-supplied metadata, so mislabeled or ambiguous environment-of-origin fields propagate into classifier training
- redbiom supports one-to-many relationships between a sample's metadata and its data across preparation types, so tag/context handling is needed when pulling metadata to avoid ambiguous sample IDs

## Evidence

- [methods] EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [methods] Redbiom is a command-line tool that allows users to query the Qiita database, including EMP studies.: "Redbiom is a command-line tool that allows users to query the Qiita database, including EMP studies."
- [supplementary] Also shown is data type (Python/Pandas), variable type, and number of the 27,751 samples having metadata for each field.: "Also shown is data type (Python/Pandas), variable type, and number of the 27,751 samples having metadata for each field."
- [readme] Redbiom is a cache service for sample metadata and sample data.: [evidence span withheld — anchor status: external_doc]
- [readme] EMP 16S Release 1, a meta-analysis of the first 97 16S rRNA amplicon studies, has been published: "EMP 16S Release 1, a meta-analysis of the first 97 16S rRNA amplicon studies, has been published"
