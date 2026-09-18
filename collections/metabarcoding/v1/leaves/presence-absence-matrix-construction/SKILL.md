---
name: presence-absence-matrix-construction
description: Use when the analysis goal is community-structure or nestedness (NODF)
  analysis on Earth Microbiome Project 16S data and you must first binarize BIOM abundance
  tables per taxonomic level or per environment subset.
when_to_use_negative:
- When the input is already a binary presence-absence table and no re-aggregation
  at a different taxonomic rank is needed
- When the analysis requires relative abundances or rarefied count data (e.g., beta
  diversity on samples rarefied to 5,000 observations) rather than qualitative incidence
  structure
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: Python
  role: Language used with IPython notebooks/scripts to load BIOM tables and mapping
    files and binarize them
- name: Pandas
  role: pandas DataFrames used to hold the BIOM-derived abundance table and mapping
    file, aggregate by taxonomic rank, and convert counts to binary values
- name: redbiom
  role: Command-line tool to query the Qiita/EMP catalogue and retrieve sample data
    into BIOM format for the target samples
  repo: github.com/biocore/redbiom
- name: QIIME2
  role: Optionally used to merge the BIOM tables and mapping files before binarization
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:05d69ab29b516fbd777d5e18abc38358a32921922574498b584d832d71ee1d37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - Pandas
  - Python
  - QIIME2
  - redbiom
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.38461538461538464
n_claims: 13
---

# presence-absence-matrix-construction

## Summary

Converts abundance/feature-count tables (e.g., EMP 16S Release 1 BIOM tables) into binary presence-absence matrices aggregated at taxonomic levels (phylum, class, order) so downstream structure metrics such as NODF nestedness can be computed.

## When to use

Use when the analysis goal is community-structure or nestedness (NODF) analysis on Earth Microbiome Project 16S data and you must first binarize BIOM abundance tables per taxonomic level or per environment subset.

## When NOT to use

- When the input is already a binary presence-absence table and no re-aggregation at a different taxonomic rank is needed
- When the analysis requires relative abundances or rarefied count data (e.g., beta diversity on samples rarefied to 5,000 observations) rather than qualitative incidence structure

## Inputs

- EMP 16S Release 1 BIOM feature table (merged, quality-filtered)
- EMP mapping/metadata file for the sample subset
- Environment-type annotations (non-saline, saline, animal, plant)

## Outputs

- Binary presence-absence matrix per taxonomic level (phylum, class, order)
- Per-environment binary presence-absence matrices (non-saline, saline, animal, plant subsets)

## How to apply

Load the EMP 16S Release 1 BIOM table and mapping file (e.g., the 2,000-sample subset) into Python with pandas, restricting to non-chloroplast taxa and applying the mc2 minimum-sequence-count filter (minimum count = 2) before aggregation. Collapse the abundance table to a chosen taxonomic rank (phylum, class, or order) and convert all counts to binary presence (1) / absence (0), producing one matrix per rank. Optionally subset rows by environment (non-saline, saline, animal, plant) using the mapping file's environment metadata before binarizing. Retrieval of the BIOM data itself can be done from the EMP catalogue with redbiom, which returns sample data into BIOM format. The rationale is that nestedness metrics such as NODF operate on decreasing-fill binary matrices, so the binarized, rank-aggregated matrix is the required substrate for the subsequent null-model permutation tests.

## Related tools

- **Python** (Language used with IPython notebooks/scripts to load BIOM tables and mapping files and binarize them)
- **Pandas** (pandas DataFrames used to hold the BIOM-derived abundance table and mapping file, aggregate by taxonomic rank, and convert counts to binary values)
- **redbiom** (Command-line tool to query the Qiita/EMP catalogue and retrieve sample data into BIOM format for the target samples) — github.com/biocore/redbiom
- **QIIME2** (Optionally used to merge the BIOM tables and mapping files before binarization)

## Examples

```
export ctx=Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506; redbiom search metadata beer | redbiom fetch samples --context $ctx --output emp_subset.biom
```

## Evaluation signals

- Every cell of the output matrix is exactly 0 or 1; no residual abundance values remain
- Chloroplast-annotated sequences are absent and OTUs below the mc2 minimum count of 2 are dropped before binarization
- The set of samples (columns) matches the mapping-file subset (e.g., the 2,000-sample EMP subset) and each matrix row corresponds to a taxon at the chosen rank (phylum/class/order)
- Per-environment matrices partition the full matrix without overlap when concatenated back (non-saline, saline, animal, plant subsets)
- Downstream NODF computed on the overall matrix falls within 0.841 +/- 0.018 with P < 0.0001, validating the binarization pipeline end-to-end

## Limitations

- Binarization discards abundance information, so results are sensitive to the choice of taxonomic aggregation level and to prior filtering (chloroplast exclusion, mc2)
- redbiom sample IDs may be tagged (<tag>_<sample-id>) and must be coerced back to valid QIIME sample IDs (<sample-id>.<tag>), otherwise joins to the mapping file can be ambiguous
- A sample ID may map to multiple redbiom IDs across preparation types; ambiguities must be resolved before binarizing

## Evidence

- [methods] EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results for ~25,000 samples in 97 studies"
- [supplementary] Sequences annotated as chloroplast were excluded before statistics were computed.: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 refers to minimum sequence count in an OTU to be included equals to 2.: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [readme] retrieval of sample data into BIOM: [evidence span withheld — anchor status: not_found]
- [readme] IMPORTANT: if you run your own resource, it is important to specify --tag on load of sample data: [evidence span withheld — anchor status: external_doc]
