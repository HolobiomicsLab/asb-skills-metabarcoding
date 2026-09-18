---
name: biom-table-parsing-and-subsetting
description: Use when the analysis needs to pull EMP sample data into a BIOM table,
  join it with mapping-file metadata (e.g., EMPO levels), and subset/rarefy samples
  so that a classifier or diversity analysis operates on a clean, comparable feature
  table.
when_to_use_negative:
- The input is already a rarefied, chloroplast-filtered feature table aligned to metadata
  — re-filtering would discard data
- The analysis needs full-depth (non-rarefied) abundance information, since the 5,000-read
  rarefaction discards deeper samples' extra reads
- The target data are not in BIOM format and cannot be retrieved through redbiom/QIIME-compatible
  processing contexts (e.g., shotgun metagenomics stored in other formats)
tools:
- name: redbiom
  role: Queries the Qiita-backed EMP catalogue by metadata or features and fetches
    sample data into BIOM within a processing context
  repo: https://github.com/biocore/redbiom
- name: Pandas
  role: Manipulates the EMP sample metadata table (data types, fields, coverage across
    the 27,751 samples) and joins metadata to the BIOM table
- name: Python
  role: Implements the metadata parsing, subsetting, and rarefaction/exclusion steps
    on the BIOM table
- name: QIIME1/QIIME2
  role: Merges BIOM tables and mapping files when combining studies into the EMP catalogue
- name: Deblur
  role: Provides the exact-sequence (90-bp V4) feature table that the parsing/subsetting
    operates on
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:7aca55cd24b683de2dd531a64a2778f6b22e5c66d07933b0bd7ae30dd623de94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - Deblur
  - Pandas
  - Python
  - QIIME1/QIIME2
  - R
  - caret
  - redbiom
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# biom-table-parsing-and-subsetting

## Summary

Load and subset BIOM feature tables and quality-filtered mapping files from the EMP 16S Release 1 catalogue (e.g., the ~25,000-sample Deblur table) using redbiom, Python/Pandas, and QIIME, applying count-based filters (rarefaction to 5,000 observations per sample, chloroplast exclusion) before downstream statistics.

## When to use

Use when the analysis needs to pull EMP sample data into a BIOM table, join it with mapping-file metadata (e.g., EMPO levels), and subset/rarefy samples so that a classifier or diversity analysis operates on a clean, comparable feature table.

## When NOT to use

- The input is already a rarefied, chloroplast-filtered feature table aligned to metadata — re-filtering would discard data
- The analysis needs full-depth (non-rarefied) abundance information, since the 5,000-read rarefaction discards deeper samples' extra reads
- The target data are not in BIOM format and cannot be retrieved through redbiom/QIIME-compatible processing contexts (e.g., shotgun metagenomics stored in other formats)

## Inputs

- Deblur BIOM feature table from EMP 16S Release 1 (~25,000 samples, 97 studies)
- Quality-filtered mapping/metadata file with EMPO level 2/3 environment annotations
- redbiom context name for the desired processing (e.g., 16S V4 Deblur context)

## Outputs

- Subset BIOM table restricted to samples with the target annotations (e.g., 2,000 EMPO-annotated samples)
- Rarefied feature table at 5,000 observations per sample with chloroplast sequences excluded
- Metadata table (mapping file) aligned with the subset sample IDs for downstream model training

## How to apply

Query the EMP catalogue with redbiom (search metadata, then fetch samples into BIOM within a processing context such as the Deblur V4 context) or load the Deblur BIOM feature table and quality-filtered mapping file directly from EMP 16S Release 1. Join the BIOM table to metadata fields (e.g., EMPO level 2/3 environment annotations) via sample IDs, and subset to samples carrying the needed annotations (e.g., ~2,000 samples with EMPO level 2/3 labels). Rarefy the table to 5,000 observations per sample so depth does not confound downstream statistics, and exclude sequences annotated as chloroplast before any statistics are computed. Write the resulting filtered table (e.g., as BIOM or a Pandas DataFrame) for use by downstream models such as the random forest classifiers in caret/randomForest.

## Related tools

- **redbiom** (Queries the Qiita-backed EMP catalogue by metadata or features and fetches sample data into BIOM within a processing context) — https://github.com/biocore/redbiom
- **Pandas** (Manipulates the EMP sample metadata table (data types, fields, coverage across the 27,751 samples) and joins metadata to the BIOM table)
- **Python** (Implements the metadata parsing, subsetting, and rarefaction/exclusion steps on the BIOM table)
- **QIIME1/QIIME2** (Merges BIOM tables and mapping files when combining studies into the EMP catalogue)
- **Deblur** (Provides the exact-sequence (90-bp V4) feature table that the parsing/subsetting operates on) — https://github.com/cuttlefishh/deblur

## Examples

```
redbiom search metadata 'empo_2' | redbiom fetch samples --context $ctx --output emp_subset.biom
```

## Evaluation signals

- The subset table contains only samples with the target EMPO level 2/3 annotations (e.g., ~2,000 samples) and every sample ID matches a mapping-file row
- After rarefaction, every sample sums to exactly 5,000 observations
- Sequences annotated as chloroplast are absent from the table before any statistics are computed
- Output is a valid BIOM table (loadable via biom-format) with sample IDs consistent with the quality-filtered mapping file
- Downstream cross-validated random forest accuracy on the subset reproduces ~91% (EMPO level 2) and ~84% (EMPO level 3), indicating the table/metadata pairing is correct

## Limitations

- Rarity/threshold handling matters: 'mc2 refers to minimum sequence count in an OTU to be included equals to 2', so features below 2 counts are dropped upstream of the table you receive
- Rarefying to 5,000 observations per sample discards samples (and reads) below or above that depth, which loses information from deeply sequenced samples
- redbiom sample IDs may carry tags (e.g., '<tag>_<sample-id>') that must be coerced back to valid QIIME-style IDs when merging across contexts

## Evidence

- [methods] EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Sequences annotated as chloroplast were excluded before statistics were computed.: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [readme] Redbiom is a cache service for sample metadata and sample data.: [evidence span withheld — anchor status: external_doc]
- [supplementary] Also shown is data type (Python/Pandas), variable type, and number of the 27,751 samples having metadata for each field.: "Also shown is data type (Python/Pandas), variable type, and number of the 27,751 samples having metadata for each field."
