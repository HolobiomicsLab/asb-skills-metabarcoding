---
name: biom-table-processing
description: Use when the input is an EMP/QIIME BIOM observation table plus a mapping
  file that must be loaded, subset by environment ontology labels, filterered (mc2,
  chloroplast), rarefied, or aggregated to taxonomic levels before statistics.
when_to_use_negative:
- The data is already a filtered, rarefied feature table with the specific analysis's
  sample subset selected — re-applying mc2/chloroplast/rarefaction filters would double-filter
- The analysis targets raw per-sample FASTQ/demultiplexed data rather than an already-built
  BIOM table; initial QIIME processing (demux, OTU picking) is a separate workflow
  step
- Shotgun metagenomic or metabolomics data, which the EMP pipeline handles with different
  SOPs rather than the 16S BIOM-table filters
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Load BIOM tables and mapping files with pandas/biom, filter, aggregate, and
    compute statistics
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Pandas
  role: Tabular handling of metadata fields and grouped summaries over the 27,751-sample
    metadata table
  repo: https://github.com/biocore/emp/tree/master/methods
- name: QIIME1
  role: Merge BIOM tables and mapping files and perform closed-reference OTU picking
    against Greengenes 13.8
  repo: https://github.com/biocore/emp/tree/master/methods
- name: redbiom
  role: Query the Qiita-backed EMP catalogue to retrieve sample data and metadata
    into BIOM for selected samples
  repo: https://github.com/biocore/redbiom
merged_aliases:
- 16s-amplicon-biom-table-processing
merged_alias_records:
- alias: 16s-amplicon-biom-table-processing
  slug: 16s-amplicon-biom-table-processing
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_008
  - task_003
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:4240d4ce19af1d59f265d29ddf50cfb3fc76ba121f4a755b3af996510fc92744
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - QIIME1
  - redbiom
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# biom-table-processing

## Summary

Load, filter, and transform BIOM-format feature tables and their mapping files from the EMP 16S Release 1 dataset using QIIME1/biom I/O and Python/Pandas, applying the release's quality filters (mc2 minimum sequence count of 2, chloroplast exclusion, rarefaction to 5,000 observations per sample) before computing richness, NODF nestedness, or PICRUSt-normalized copy-number statistics.

## When to use

Use when the input is an EMP/QIIME BIOM observation table plus a mapping file that must be loaded, subset by environment ontology labels, filterered (mc2, chloroplast), rarefied, or aggregated to taxonomic levels before statistics.

## When NOT to use

- The data is already a filtered, rarefied feature table with the specific analysis's sample subset selected — re-applying mc2/chloroplast/rarefaction filters would double-filter
- The analysis targets raw per-sample FASTQ/demultiplexed data rather than an already-built BIOM table; initial QIIME processing (demux, OTU picking) is a separate workflow step
- Shotgun metagenomic or metabolomics data, which the EMP pipeline handles with different SOPs rather than the 16S BIOM-table filters

## Inputs

- EMP 16S Release 1 merged BIOM feature/observation table (Deblur exact-sequence or closed-reference Greengenes 13.8, 16S V4)
- Merged sample metadata mapping file with EMPO/environment ontology annotations
- Closed-reference BIOM observation table for PICRUSt copy-number normalization

## Outputs

- Filtered/aggregated BIOM or presence-absence matrices (phylum/class/order levels)
- Per-sample statistics tables: empo_richness_summary.csv, nodf_results.csv, acn_per_sample.tsv
- Summary tables keyed by environment/EMPO category with computed statistics

## How to apply

Load the merged BIOM feature table and mapping file with Python (pandas/biom) or QIIME1/biom I/O, as in the EMP 16S Release 1 pipeline covering ~23,828 analyzed samples from ~25,000 in the release. Apply the release's stated filters before any statistics: exclude sequences annotated as chloroplast, enforce mc2 (minimum sequence count in an OTU equals 2), and confirm samples are rarefied to 5,000 observations per sample for depth-normalized richness comparisons. For downstream analyses, convert abundance tables to binary presence-absence matrices aggregated at phylum/class/order levels (NODF nestedness) or run closed-reference Greengenes 13.8 tables through PICRUSt normalize_by_copy_number.py (ACN analysis). Decide on the observation unit — exact Deblur sequences for richness, closed-reference Greengenes OTU IDs for PICRUSt — based on the analysis target, since the release provides both Deblur and closed-reference picking. Write per-analysis outputs (e.g., nodf_results.csv, acn_per_sample.tsv, empo_richness_summary.csv) with one row per group or sample plus the computed statistic.

## Related tools

- **Python** (Load BIOM tables and mapping files with pandas/biom, filter, aggregate, and compute statistics) — https://github.com/biocore/emp/tree/master/methods
- **Pandas** (Tabular handling of metadata fields and grouped summaries over the 27,751-sample metadata table) — https://github.com/biocore/emp/tree/master/methods
- **QIIME1** (Merge BIOM tables and mapping files and perform closed-reference OTU picking against Greengenes 13.8) — https://github.com/biocore/emp/tree/master/methods
- **redbiom** (Query the Qiita-backed EMP catalogue to retrieve sample data and metadata into BIOM for selected samples) — https://github.com/biocore/redbiom

## Examples

```
python -c "import biom, pandas as pd; t = biom.load_table('emp_deblur.biom'); md = pd.read_csv('emp_mapping.tsv', sep='\t'); md = md[md[' sequences chloroplast'] == 0]"
```

## Evaluation signals

- The BIOM table reflects the release filters: chloroplast-annotated sequences excluded, mc2 minimum sequence count of 2 applied, and ~23,828 of ~25,000 samples retained
- Alpha-diversity comparisons use samples rarefied to exactly 5,000 observations per sample so richness is depth-normalized
- Grouped summary outputs contain one row per EMPO/environment category (e.g., free-living vs host-associated) with a computed statistic, matching the Fig. 2a ordering
- PICRUSt-normalized ACN per-sample table centers free-living/plant-associated samples near 2.2 and animal-associated samples near 3.4
- Presence-absence matrices aggregated at phylum/class/order levels yield overall NODF within 0.841 ± 0.018 with P < 0.05 versus the equiprobable-rows/fixed-columns null model

## Limitations

- The mc2 filter and chloroplast exclusion are specific to this 16S release; applying them to other datasets assumes the same quality-control conventions
- Redbiom's default host qiita.ucsd.edu:7329 is read-only; administrative data loading requires a local Redis/Webdis instance and an explicit REDBIOM_HOST setting
- Sample IDs in redbiom contexts may be tag-prefixed (<tag>_<sample-id>) and must be coerced to valid QIIME IDs (<sample-id>.<tag>), with ambiguities flagged on write

## Evidence

- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] chloroplast exclusion before statistics: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] QIIME merging of BIOM tables: [evidence span withheld — anchor status: not_found]
- [readme] redbiom as cache service for sample data into BIOM: [evidence span withheld — anchor status: external_doc]
- [methods] release contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information"
