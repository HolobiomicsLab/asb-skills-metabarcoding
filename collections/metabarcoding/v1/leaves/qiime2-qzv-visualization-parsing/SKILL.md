---
name: qiime2-qzv-visualization-parsing
description: Use when a QIIME 2 run's .qzv outputs (e.g., observed_features_group_significance.qzv,
  alpha_rarefaction.qzv) must be parsed to extract per-sample values or curve data
  for scripted validation or downstream analysis.
when_to_use_negative:
- When the diversity comparison can be read directly from the interactive HTML report
  or QIIME 2 viewer and no scripted verification or downstream join with metadata
  is needed
- When the input is a QIIME 2 data artifact (.qza, e.g., a feature table) rather than
  a visualization (.qzv) — use artifact loading, not visualization parsing
- When rarefaction depth parameters differ (the ~350 sequences/sample plateau expectation
  is specific to runs with alpha_max_depth 500 and core_sampling_depth 500)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: QIIME 2
  role: Produces the .qzv visualizations and provides the Python API (Artifact API)
    used to load and parse them programmatically
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2 viewer
  role: Interactive web viewer for the .qzv files, used as an alternative to scripted
    parsing for eyeballing results
- name: Tourmaline
  role: Workflow that generates the diversity outputs (group significance .qzv, alpha_rarefaction.qzv)
    and configures rarefaction depth parameters
  repo: https://github.com/aomlomics/tourmaline
- name: Jupyter notebooks
  role: Python-based notebooks provided with Tourmaline that consume QIIME 2 artifact
    files for downstream analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:691490e94994e59f99a9e25c34cd7e25c90091710046efc82df95060f80cf88b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Jupyter notebooks
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Tourmaline
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# qiime2-qzv-visualization-parsing

## Summary

Programmatically extract data embedded in QIIME 2 visualization files (.qzv), such as per-sample observed-features values and alpha-rarefaction curves, using the QIIME 2 Artifact API and pandas. This enables verification of diversity findings (e.g., group comparisons, saturation depth) that are normally only viewable interactively.

## When to use

Use when a QIIME 2 run's .qzv outputs (e.g., observed_features_group_significance.qzv, alpha_rarefaction.qzv) must be parsed to extract per-sample values or curve data for scripted validation or downstream analysis.

## When NOT to use

- When the diversity comparison can be read directly from the interactive HTML report or QIIME 2 viewer and no scripted verification or downstream join with metadata is needed
- When the input is a QIIME 2 data artifact (.qza, e.g., a feature table) rather than a visualization (.qzv) — use artifact loading, not visualization parsing
- When rarefaction depth parameters differ (the ~350 sequences/sample plateau expectation is specific to runs with alpha_max_depth 500 and core_sampling_depth 500)

## Inputs

- QIIME 2 visualization file (.qzv), e.g., observed_features_group_significance.qzv from the diversity outputs of a Tourmaline (unfiltered) run
- QIIME 2 alpha rarefaction visualization (alpha_rarefaction.qzv) generated with plot_diversity: true, alpha_max_depth 500, core_sampling_depth 500
- Sample metadata TSV file (e.g., 00-data/metadata.tsv) with a size-fraction column (5.0-um vs 0.22-um)

## Outputs

- Per-sample observed features values stratified by metadata group
- Rarefaction curve plateau depth (sequences per sample)
- Compact verification table (alpha_diversity_verification.tsv) with group medians, direction of difference, plateau depth, and pass/fail flags

## How to apply

Load the .qzv file with the QIIME 2 Artifact API (Visualization.load), then extract the underlying per-sample data into pandas DataFrames — for the group significance .qzv, pull per-sample observed features; for alpha_rarefaction.qzv, pull the observed-features rarefaction curves. Join the extracted per-sample values with the sample metadata file (e.g., 00-data/metadata.tsv) to stratify samples by metadata groups such as the 5.0-um and 0.22-um size fractions. Compare richness between groups (medians, direction of difference) and identify the sampling depth where rarefaction curves plateau (the Tourmaline config generated these with alpha_max_depth 500 and core_sampling_depth 500, with plateau expected near ~350 sequences/sample). Write the comparison as a compact summary table, e.g., alpha_diversity_verification.tsv with group medians, direction, plateau depth, and pass/fail flags. This follows the Tourmaline design in which QIIME 2 artifact files can be fed directly into Python- and R-based Jupyter notebooks.

## Related tools

- **QIIME 2** (Produces the .qzv visualizations and provides the Python API (Artifact API) used to load and parse them programmatically) — https://github.com/aomlomics/tourmaline
- **QIIME 2 viewer** (Interactive web viewer for the .qzv files, used as an alternative to scripted parsing for eyeballing results)
- **Tourmaline** (Workflow that generates the diversity outputs (group significance .qzv, alpha_rarefaction.qzv) and configures rarefaction depth parameters) — https://github.com/aomlomics/tourmaline
- **Jupyter notebooks** (Python-based notebooks provided with Tourmaline that consume QIIME 2 artifact files for downstream analysis)

## Examples

```
from qiime2 import Visualization; import pandas as pd; v = Visualization.load('alpha_rarefaction.qzv')
```

## Evaluation signals

- Per-sample observed-features values extracted from the .qzv match the values visible in the QIIME 2 viewer / HTML report
- Joined group comparison shows the expected direction: the 5.0-um fraction has higher observed features than the 0.22-um fraction
- Rarefaction curves extracted from alpha_rarefaction.qzv level off by approximately 350 sequences per sample
- alpha_diversity_verification.tsv contains group medians, direction of difference, plateau depth, and pass/fail flags with no missing metadata joins
- All .qzv files parse without provenance/API errors under the QIIME 2 Artifact API

## Limitations

- Parsing .qzv via the Artifact API requires a compatible QIIME 2 version; visualization formats are tied to the QIIME 2 version that produced them
- Plateau depth conclusions depend on the configured rarefaction range (alpha_max_depth 500 in the Tourmaline config); shallower or deeper max depths change the saturation assessment
- Group comparisons are stratified by the metadata column supplied (size fraction); mis-specified metadata or sample IDs will break the join or mislabel groups

## Evidence

- [intro] QIIME 2, with its built-in provenance tracking, archive format, interactive visualizations, multiple interfaces including a Python API: "QIIME 2, with its built-in provenance tracking, archive format, interactive visualizations, multiple interfaces including a Python API"
- [intro] QIIME 2 artifact files can be fed directly into provided Python- and R-based Jupyter notebooks: "QIIME 2 artifact files can be fed directly into provided Python- and R-based Jupyter notebooks"
- [abstract] the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [abstract] performs analyses of taxonomic, alpha, and beta diversity: "performs analyses of taxonomic, alpha, and beta diversity"
