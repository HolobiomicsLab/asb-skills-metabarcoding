---
name: microbial-community-richness-analysis
description: Use when a QIIME 2/Tourmaline run has produced alpha-diversity outputs
  (observed_features_group_significance.qzv, alpha_rarefaction.qzv) and you need to
  compare richness across a metadata grouping or find the rarefaction plateau depth.
when_to_use_negative:
- The input is already a feature table or per-sample richness vector rather than a
  QIIME 2 .qzv visualization — extract values directly instead of re-parsing.
- The question concerns beta diversity (between-sample community structure) rather
  than within-sample richness; Tourmaline's alpha and beta diversity analyses are
  separate outputs.
- Rarefaction curves have not plateaued within the configured alpha_max_depth (e.g.
  curves still rising at 500 sequences/sample), so the observed-features comparison
  would be depth-confounded.
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Produces the alpha-diversity group-significance and rarefaction visualizations
    (.qzv) that are parsed for richness comparison
- name: Tourmaline
  role: Containerized Snakemake workflow that runs the QIIME 2 alpha diversity analyses
    and emits the outputs consumed here
  repo: https://github.com/aomlomics/tourmaline
- name: Python
  role: Used via the QIIME 2 Artifact API and pandas to parse .qzv files and compute
    group medians
- name: Jupyter notebooks
  role: Downstream notebooks provided with Tourmaline that consume QIIME 2 artifact
    files for analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:46a1446fa11dd4274b6d77917d133f727e36fde2bdc82c6532a78d7b2ec4da4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Jupyter notebooks
  - Python
  - QIIME 2
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# microbial-community-richness-analysis

## Summary

Compares within-sample alpha diversity (observed features) between sample groups and identifies the sequencing depth at which richness saturates, by parsing QIIME 2 alpha-diversity visualizations produced by the Tourmaline workflow.

## When to use

Use when a QIIME 2/Tourmaline run has produced alpha-diversity outputs (observed_features_group_significance.qzv, alpha_rarefaction.qzv) and you need to compare richness across a metadata grouping or find the rarefaction plateau depth.

## When NOT to use

- The input is already a feature table or per-sample richness vector rather than a QIIME 2 .qzv visualization — extract values directly instead of re-parsing.
- The question concerns beta diversity (between-sample community structure) rather than within-sample richness; Tourmaline's alpha and beta diversity analyses are separate outputs.
- Rarefaction curves have not plateaued within the configured alpha_max_depth (e.g. curves still rising at 500 sequences/sample), so the observed-features comparison would be depth-confounded.

## Inputs

- observed_features_group_significance.qzv (QIIME 2 alpha group significance visualization from the diversity outputs)
- alpha_rarefaction.qzv (QIIME 2 rarefaction visualization, generated with plot_diversity: true, alpha_max_depth 500, core_sampling_depth 500)
- sample metadata file (e.g. 00-data/metadata.tsv) with the size-fraction or other grouping column

## Outputs

- alpha_diversity_verification.tsv containing group medians, direction of difference, plateau depth, and pass/fail flags
- per-sample observed-features values stratified by group

## How to apply

Parse the QIIME 2 group significance visualization observed_features_group_significance.qzv with the QIIME 2 Artifact/Visualization API and pandas to extract per-sample observed-features values, then join these to the sample metadata file (e.g. 00-data/metadata.tsv) to stratify samples by the grouping of interest, such as the 5.0-um vs 0.22-um size fractions in the Lake Erie 16S rRNA dataset. Compute group medians and the direction of difference in richness. Next parse alpha_rarefaction.qzv to extract observed-features rarefaction curves (generated with plot_diversity: true, alpha_max_depth 500, core_sampling_depth 500) and identify the depth at which curves plateau — e.g. ~350 sequences/sample. QIIME 2 artifact files can be fed directly into the Python- and R-based Jupyter notebooks provided with Tourmaline for this downstream analysis.

## Related tools

- **QIIME 2** (Produces the alpha-diversity group-significance and rarefaction visualizations (.qzv) that are parsed for richness comparison)
- **Tourmaline** (Containerized Snakemake workflow that runs the QIIME 2 alpha diversity analyses and emits the outputs consumed here) — https://github.com/aomlomics/tourmaline
- **Python** (Used via the QIIME 2 Artifact API and pandas to parse .qzv files and compute group medians)
- **Jupyter notebooks** (Downstream notebooks provided with Tourmaline that consume QIIME 2 artifact files for analysis)

## Examples

```
from qiime2 import Visualization; Visualization.load('observed_features_group_significance.qzv').view(pd.DataFrame)
```

## Evaluation signals

- The 5.0-um fraction shows higher observed features than the 0.22-um fraction across the group-significance results.
- Rarefaction curves level off by approximately 350 sequences per sample, matching the expected plateau depth.
- alpha_diversity_verification.tsv contains group medians, direction of difference, plateau depth, and pass/fail flags for every group.
- Per-sample values extracted from the .qzv join cleanly (one-to-one) against sample IDs in 00-data/metadata.tsv.

## Limitations

- Richness comparisons are only valid up to the rarefaction plateau (~350 sequences/sample in the Lake Erie test set); below saturation, differences may reflect sampling depth rather than true diversity.
- Configured depth caps (alpha_max_depth 500, core_sampling_depth 500) limit the resolvable saturation point; shallower datasets or deeper sequencing require re-running with different config parameters.

## Evidence

- [abstract] Taxonomic, alpha, and beta diversity analyses: "performs analyses of taxonomic, alpha, and beta diversity"
- [intro] Jupyter notebooks consume QIIME 2 artifacts: "QIIME 2 artifact files can be fed directly into provided Python- and R-based Jupyter notebooks"
- [abstract] Tourmaline demonstrated on Lake Erie 16S data with interactive visualization: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
