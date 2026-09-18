---
name: biom-table-merging-across-studies
description: Use when per-study BIOM tables (demultiplexed, Deblur-processed, QC-filtered)
  must be combined into one merged feature table for cross-study meta-analysis, e.g.,
  building emp_deblur_90bp_qc.biom.
when_to_use_negative:
- Input data are clustered OTU tables rather than exact-sequence (Deblur) tables —
  cross-study sequence matching relies on exact sequences instead of clustered OTUs
- Per-study tables have not yet passed the mc2 filter and >=25-read sample retention,
  so merging would propagate low-quality features
- Studies use different amplicon regions (e.g., non-V4 or non-90-bp trimmed sequences),
  so sequences are not comparable after merging
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Deblur
  role: exact-sequence de novo OTU picking method applied to demultiplexed reads,
    trimmed to 90 bp, before per-study BIOM tables are produced
  repo: https://github.com/cuttlefishh/deblur
- name: QIIME1
  role: used to merge the per-study BIOM tables and mapping files
- name: QIIME2
  role: alternative tool to merge the BIOM tables and mapping files
- name: Python
  role: scripts and IPython notebooks used in the meta-analysis workflow for validation
    and table handling
  repo: https://github.com/biocore/emp/tree/master/methods
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:6db805d890360b8b8b3df44b9f74c480be9cb7ddf97ba0a280e1e44ca60d0a0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Python
  - QIIME1
  - QIIME2
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 6
---

# biom-table-merging-across-studies

## Summary

Merges per-study QC-filtered BIOM feature tables (e.g., Deblur 90-bp tag-sequence tables from EMP 16S Release 1) into a single cross-study observation table using QIIME1 or QIIME2, preserving exact sequences so bacterial and archaeal rRNA sequences can be followed across studies.

## When to use

Use when per-study BIOM tables (demultiplexed, Deblur-processed, QC-filtered) must be combined into one merged feature table for cross-study meta-analysis, e.g., building emp_deblur_90bp_qc.biom.

## When NOT to use

- Input data are clustered OTU tables rather than exact-sequence (Deblur) tables — cross-study sequence matching relies on exact sequences instead of clustered OTUs
- Per-study tables have not yet passed the mc2 filter and >=25-read sample retention, so merging would propagate low-quality features
- Studies use different amplicon regions (e.g., non-V4 or non-90-bp trimmed sequences), so sequences are not comparable after merging

## Inputs

- per-study demultiplexed fastq files (demux tables) from the EMP 16S Release 1 FTP deposit (ftp://ftp.microbio.me/emp/release1)
- per-study QC-filtered Deblur 90-bp BIOM feature tables
- per-study mapping files

## Outputs

- merged BIOM feature table (e.g., emp_deblur_90bp_qc.biom) spanning 96 studies and 23,828 samples with 307,572 unique 90-bp tag sequences after rarefaction to 5,000 per sample
- merged mapping/metadata file

## How to apply

First generate per-study BIOM tables by splitting raw reads into per-sample fastq files by barcode (demux), running Deblur on the demultiplexed reads trimmed to 90 bp to match the V4 tag length, then applying the mc2 filter (minimum sequence count of 2 per OTU) and retaining only samples with >=25 total reads, and rarefying to 5,000 observations per sample. Then use QIIME1 or QIIME2 to merge the per-study BIOM tables and mapping files into a single table. Rationale: exact sequences rather than clustered OTUs are used so identical V4 tag sequences align across studies. Validate the merged table against expected counts before saving it (e.g., emp_deblur_90bp_qc.biom with 96 studies, 23,828 samples, 307,572 unique sequences after rarefaction to 5,000 per sample).

## Related tools

- **Deblur** (exact-sequence de novo OTU picking method applied to demultiplexed reads, trimmed to 90 bp, before per-study BIOM tables are produced) — https://github.com/cuttlefishh/deblur
- **QIIME1** (used to merge the per-study BIOM tables and mapping files)
- **QIIME2** (alternative tool to merge the BIOM tables and mapping files)
- **Python** (scripts and IPython notebooks used in the meta-analysis workflow for validation and table handling) — https://github.com/biocore/emp/tree/master/methods

## Examples

```
merge_otu_tables.py -i study1.biom,study2.biom,...,study96.biom -o emp_deblur_90bp_qc.biom
```

## Evaluation signals

- Merged table contains the expected 96 studies and 23,828 samples with 307,572 unique sequences after rarefying to 5,000 observations per sample
- Sum of samples across per-study BIOM tables equals the sample count of the merged table (no lost or duplicated sample IDs)
- Unique sequence (feature) IDs in the merged table are all exactly 90 bp, consistent with V4 tag-length trimming
- All features appear in at least 2 samples (mc2 filter) and every retained sample has >=25 total reads before rarefaction
- Merged BIOM table is loadable by BIOM/QIIME tooling (BIOM >= 2.1.5) with intact observation and sample metadata

## Limitations

- EM P Release 1 spans only the first 97 16S rRNA amplicon studies (~25,000 samples); later releases (EMP 16S Release 2, EMP500) use additional data and workflows
- Ambiguous sample IDs can arise when merging tables whose sample IDs are not mutually exclusive; tags or prefixes may be needed to disambiguate
- Studies with fewer than 25 total reads per sample are dropped, so the merged table underrepresents very low-depth studies

## Evidence

- [methods] Use QIIME1 or QIIME2 to merge BIOM tables: [evidence span withheld — anchor status: not_found]
- [supplementary] Samples were rarefied to 5,000 observations per sample: "Samples were rarefied to 5,000 observations per sample."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [abstract] Exact sequences enable cross-study comparison: "Coordinated protocols and new analytical methods, particularly the use of exact sequences instead of clustered operational taxonomic units, enable bacterial and archaeal ribosomal RNA gene sequences"
- [methods] Release 1 contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
