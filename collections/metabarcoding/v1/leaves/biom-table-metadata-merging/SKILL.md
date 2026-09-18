---
name: biom-table-metadata-merging
description: Use when you need to combine multiple BIOM tables and sample metadata
  (mapping files) from a meta-analysis — e.g., before computing habitat-specificity
  or entropy statistics over EMP tag sequences across ~97 studies.
when_to_use_negative:
- You only need to retrieve or search existing EMP data rather than merge local tables
  — use redbiom queries against the Qiita database instead
- The analysis operates on a single study whose BIOM table and mapping file are already
  paired — merging adds no information
- You need per-study (non-pooled) results, where cross-study merging would obscure
  study-level structure
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QIIME1
  role: Merges the BIOM tables and mapping files; its biom tooling converts BIOM objects
    during the merge
- name: QIIME2
  role: Alternative to QIIME1 for merging BIOM tables and mapping files
- name: redbiom
  role: Fetches sample data and sample metadata into BIOM/mapping form from the Qiita-hosted
    EMP catalogue prior to merging
  repo: https://github.com/biocore/redbiom
- name: Pandas
  role: Used with Python to hold the merged table and mapping fields for downstream
    entropy computation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:465e5028e7991bc95c4132ca98f2b28fd7c60fec0593fd2ac0fd01f52c721a04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - Pandas
  - Python
  - QIIME1
  - QIIME2
  - redbiom
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# biom-table-metadata-merging

## Summary

Merges multiple BIOM feature tables (e.g., EMP 16S Release 1 Deblur tag-sequence and closed-reference OTU tables) with quality-filtered sample mapping files using QIIME1/QIIME2 biom tooling, producing a single metadata-annotated table ready for downstream Python/Pandas analyses such as per-tag-sequence Shannon entropy of environment distributions.

## When to use

Use when you need to combine multiple BIOM tables and sample metadata (mapping files) from a meta-analysis — e.g., before computing habitat-specificity or entropy statistics over EMP tag sequences across ~97 studies.

## When NOT to use

- You only need to retrieve or search existing EMP data rather than merge local tables — use redbiom queries against the Qiita database instead
- The analysis operates on a single study whose BIOM table and mapping file are already paired — merging adds no information
- You need per-study (non-pooled) results, where cross-study merging would obscure study-level structure

## Inputs

- EMP 16S Release 1 Deblur tag-sequence BIOM table (90-bp V4 sequences, starting after primer 515f)
- Closed-reference OTU BIOM tables (picked against Greengenes 13.8 or Silva 123)
- Merged quality-filtered mapping files (~25,000 samples in 97 studies; 27,751 samples with metadata fields)

## Outputs

- Merged BIOM feature table joined with sample metadata (mapping-file columns such as environment category)
- Filtered/rarefied table (5,000 observations per sample, chloroplast sequences excluded, mc2 filter applied) suitable for per-feature entropy calculations

## How to apply

Download the BIOM tables and merged quality-filtered mapping files (e.g., EMP 16S Release 1 from ftp://ftp.microbio.me/emp/release1), converting BIOM objects via QIIME1 biom tooling. Use QIIME1 or QIIME2 to merge the BIOM tables and mapping files so each sample in the table carries its metadata categories (e.g., environment, pH). Follow the EMP preprocessing conventions: rarefy to 5,000 observations per sample, exclude sequences annotated as chloroplast, and apply the mc2 filter (minimum sequence count in an OTU equals 2) before computing per-feature statistics. In redbiom-backed contexts, watch for sample IDs prefixed by a tag (of the form <tag>_<sample-id>), since the same sample may exist under multiple processing contexts (e.g., Deblur vs. closed-reference against Greengenes 13.8). Validate the merge by checking that sample identifiers in the merged table match the mapping file and that the row count corresponds to the expected feature type (exact 90-bp V4 Deblur tag sequences or Greengenes OTU IDs).

## Related tools

- **QIIME1** (Merges the BIOM tables and mapping files; its biom tooling converts BIOM objects during the merge)
- **QIIME2** (Alternative to QIIME1 for merging BIOM tables and mapping files)
- **redbiom** (Fetches sample data and sample metadata into BIOM/mapping form from the Qiita-hosted EMP catalogue prior to merging) — https://github.com/biocore/redbiom
- **Pandas** (Used with Python to hold the merged table and mapping fields for downstream entropy computation)

## Evaluation signals

- Every sample ID in the merged BIOM table resolves to exactly one row in the merged quality-filtered mapping file (no dropped or duplicated samples)
- After rarefaction, each retained sample has exactly 5,000 observations, matching the EMP supplementary protocol
- No sequences annotated as chloroplast remain, and no OTU has a minimum sequence count below 2 (mc2 filter respected)
- Feature IDs have the expected form: exact 90-bp V4 Deblur tag sequences, or Greengenes OTU IDs (e.g., 943389) for the closed-reference context
- Downstream sanity check: every Shannon entropy value computed from the merged table lies between 0 and log2(number of environment categories)

## Limitations

- Sample IDs can be ambiguous across processing contexts; redbiom requires a --tag on load because sample IDs between tables 'may not be mutually exclusive'
- Only samples with metadata can be retained — 27,751 samples have metadata fields out of the broader EMP catalogue
- The default redbiom host is read-only, so data cannot be loaded into the shared cache without a local Redis/Webdis instance

## Evidence

- [methods] QIIME1 merge role: [evidence span withheld — anchor status: not_found]
- [supplementary] rarefaction depth: "Samples were rarefied to 5,000 observations per sample."
- [methods] mc2 filter definition: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] chloroplast exclusion: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [readme] table ID ambiguity across contexts: [evidence span withheld — anchor status: external_doc]
