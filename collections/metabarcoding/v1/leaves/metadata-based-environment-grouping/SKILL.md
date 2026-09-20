---
name: metadata-based-environment-grouping
description: Use when a meta-analysis feature table is accompanied by a merged metadata
  mapping file with environment annotations (e.g., EMPO levels) and you need to aggregate
  per-sample statistics, such as observed tag-sequence richness, by environment category.
when_to_use_negative:
- When the goal is per-sample or per-study diversity without environment-level aggregation
  — the EMPO grouping is the point of this skill
- When the feature table is not depth-normalized (not rarefied to 5,000 observations
  per sample) — raw richness comparisons across environments would be biased
- When metadata lack standardized environment ontology annotations (EMPO or equivalent)
  — free-text habitats cannot be grouped reproducibly
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Python
  role: Scripting environment for loading the BIOM table, joining EMPO metadata, and
    computing per-sample richness
- name: Pandas
  role: DataFrame groupby aggregation of per-sample richness by EMPO environment category
    and CSV export of the summary table
- name: redbiom
  role: Command-line retrieval of sample metadata and sample data (into BIOM) from
    the Qiita-backed EMP catalogue, including pulling sample metadata tied to a processing
    context
  repo: https://github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:66972a075346ad2b45a43210284329c55de79a118370a0ae7de1cd31c7e12c37
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biocore/redbiom
  tools:
  - Pandas
  - Python
  - redbiom
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# metadata-based-environment-grouping

## Summary

Groups per-sample alpha-diversity results by standardized environment ontology categories (e.g., EMPO free-living vs. host-associated) from the EMP merged mapping file, enabling environment-level comparisons such as the Fig. 2a richness contrast in EMP 16S Release 1.

## When to use

Use when a meta-analysis feature table is accompanied by a merged metadata mapping file with environment annotations (e.g., EMPO levels) and you need to aggregate per-sample statistics, such as observed tag-sequence richness, by environment category.

## When NOT to use

- When the goal is per-sample or per-study diversity without environment-level aggregation — the EMPO grouping is the point of this skill
- When the feature table is not depth-normalized (not rarefied to 5,000 observations per sample) — raw richness comparisons across environments would be biased
- When metadata lack standardized environment ontology annotations (EMPO or equivalent) — free-text habitats cannot be grouped reproducibly

## Inputs

- Rarefied, quality-filtered Deblur BIOM feature table from EMP 16S Release 1 (5,000 observations per sample, mc2, chloroplast sequences excluded)
- Merged metadata mapping file carrying EMPO environment annotations (sample IDs as index)

## Outputs

- empo_richness_summary.csv — one row per EMPO environment category with per-group richness statistics (median/mean observed tag-sequence richness)
- Per-sample observed richness vector (count of distinct exact-sequence features per sample)

## How to apply

Load the rarefied, quality-filtered Deblur BIOM table (~23,828 analyzed samples from the ~25,000 in EMP 16S Release 1) together with the merged mapping file using Python with pandas and the biom library. Confirm the release filters apply: mc2 (minimum sequence count in an OTU equals 2) and exclusion of chloroplast-annotated sequences before statistics, and that samples were rarefied to 5,000 observations per sample so richness comparisons are depth-normalized. Compute observed tag-sequence richness per sample as the count of distinct exact-sequence features present, then join the mapping file's EMPO annotation and group richness by environment category using pandas groupby. Decide the grouping level (e.g., free-living vs. host-associated, or finer EMPO levels such as plant rhizosphere) before computing summary statistics (median/mean) per group; write the grouped summary to a CSV such as empo_richness_summary.csv.

## Related tools

- **Python** (Scripting environment for loading the BIOM table, joining EMPO metadata, and computing per-sample richness)
- **Pandas** (DataFrame groupby aggregation of per-sample richness by EMPO environment category and CSV export of the summary table)
- **redbiom** (Command-line retrieval of sample metadata and sample data (into BIOM) from the Qiita-backed EMP catalogue, including pulling sample metadata tied to a processing context) — https://github.com/biocore/redbiom

## Examples

```
df = mapping.join(richness); df.groupby('empo_3')['observed_richness'].describe().to_csv('empo_richness_summary.csv')
```

## Evaluation signals

- The summary table contains exactly one row per EMPO environment category, each with a richness statistic
- Free-living groups rank above host-associated groups in median/mean richness, except plant rhizosphere which resembles free-living soil — reproducing the published Fig. 2a ordering
- Sample count reconciles with the release: ~23,828 analyzed samples of the ~25,000 in EMP 16S Release 1
- Per-sample richness values are consistent with 5,000 observations-per-sample rarefaction (no sample exceeds 5,000 total observations)
- Chloroplast-annotated sequences and OTUs failing mc2 are absent from the richness counts

## Limitations

- Results depend on the release filters (mc2, chloroplast exclusion, 5,000-observation rarefaction); applying the grouping to a differently processed table will not reproduce the published ordering
- The 16S Release 1 catalogue covers only the first 97 16S rRNA amplicon studies (~25,000 samples); environment categories with few samples in a given EMPO class may have unstable summary statistics
- EMPO annotations come from the merged mapping file supplied with the release; reclassifying samples requires regenerating or validating those metadata categories separately

## Evidence

- [methods] mc2 refers to minimum sequence count in an OTU to be included equals to 2.: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] rarefied to 5,000: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] chloroplast exclusion: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] Release 1 contents: "EMP 16S Release 1 contains merged and quality-filtered mapping files, BIOM tables, OTU/sequence information, and alpha/beta-diversity results"
- [readme] redbiom metadata retrieval: [evidence span withheld — anchor status: external_doc]
