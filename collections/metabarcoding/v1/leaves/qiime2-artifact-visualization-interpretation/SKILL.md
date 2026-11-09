---
name: qiime2-artifact-visualization-interpretation
description: Use when QIIME 2 distance matrices and ordinations exist (or a .qzv report
  is produced) and you must inspect Emperor plots or extract PERMANOVA pseudo-F/p-values
  to judge whether samples group by a metadata column.
when_to_use_negative:
- You only need taxonomic composition or alpha diversity — this skill addresses beta
  diversity ordination and PERMANOVA separation tests specifically
- The feature table or representative-sequence artifacts are missing or a new run
  has not yet produced them (Tourmaline will not emit the diversity .qzv outputs)
- The metadata file lacks the grouping columns (e.g. location or size fraction) needed
  by beta-group-significance
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0091
tools:
- name: QIIME 2
  role: Computes UniFrac distance matrices, PCoA ordinations (core-metrics at core_sampling_depth
    500), Emperor plots, and PERMANOVA beta-group-significance tests
- name: Tourmaline
  role: Snakemake workflow that runs the QIIME 2 diversity/visualization steps and
    emits the .qzv files via its HTML report
  repo: github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system Tourmaline uses to orchestrate the QIIME 2 commands
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2 viewer
  role: Used to view the interactive .qzv visualizations (Emperor ordinations, beta-group-significance
    plots) linked from the HTML report
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:1d60ae670a28af12483d2148307c957db285132b6532cbf568e9876052e1013b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# qiime2-artifact-visualization-interpretation

## Summary

Generate and interpret QIIME 2 visualization artifacts (.qzv) — e.g. unweighted UniFrac Emperor ordinations and beta-group-significance (PERMANOVA) plots — to test whether 16S communities separate by metadata factors such as location and size fraction.

## When to use

Use when QIIME 2 distance matrices and ordinations exist (or a .qzv report is produced) and you must inspect Emperor plots or extract PERMANOVA pseudo-F/p-values to judge whether samples group by a metadata column.

## When NOT to use

- You only need taxonomic composition or alpha diversity — this skill addresses beta diversity ordination and PERMANOVA separation tests specifically
- The feature table or representative-sequence artifacts are missing or a new run has not yet produced them (Tourmaline will not emit the diversity .qzv outputs)
- The metadata file lacks the grouping columns (e.g. location or size fraction) needed by beta-group-significance

## Inputs

- [run_name]-table.qza (QIIME 2 feature table artifact)
- [run_name]-repseqs.qza (representative sequences artifact)
- sample metadata file mapping samples to location (Open Water vs. Western Boundary) and size fraction (0.22-um vs. 5.0-um)
- core_sampling_depth setting (500) from the Tourmaline config when plot_diversity: true

## Outputs

- unweighted_unifrac_emperor.qzv (Emperor ordination visualization)
- beta_group_significance.qzv (PERMANOVA results for each factor/metric combination)
- beta_group_significance_results.tsv (grouping factor, distance metric, pseudo-F, p-value, separation verdict)

## How to apply

Load the run's [run_name]-table.qza and [run_name]-repseqs.qza together with the sample metadata file mapping each sample to location (Open Water vs. Western Boundary) and size fraction (0.22-um vs. 5.0-um). Compute UniFrac distance matrices and PCoA ordinations with the QIIME 2 core-metrics workflow at core_sampling_depth: 500 (the depth configured when plot_diversity: true). Generate the unweighted_unifrac_emperor.qzv ordination and inspect it colored by location and by size fraction, then run qiime diversity beta-group-significance (PERMANOVA) on the unweighted and weighted UniFrac matrices grouped by each factor. Extract the pseudo-F statistic and p-value from each beta_group_significance.qzv and record a separation verdict per factor (p < 0.05). Write the grouping factor, distance metric, pseudo-F, p-value, and verdict to beta_group_significance_results.tsv.

## Related tools

- **QIIME 2** (Computes UniFrac distance matrices, PCoA ordinations (core-metrics at core_sampling_depth 500), Emperor plots, and PERMANOVA beta-group-significance tests)
- **Tourmaline** (Snakemake workflow that runs the QIIME 2 diversity/visualization steps and emits the .qzv files via its HTML report) — github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system Tourmaline uses to orchestrate the QIIME 2 commands) — github.com/aomlomics/tourmaline
- **QIIME 2 viewer** (Used to view the interactive .qzv visualizations (Emperor ordinations, beta-group-significance plots) linked from the HTML report)

## Examples

```
qiime diversity beta-group-significance --i-distance-matrix [run_name]-unweighted_unifrac_distance_matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column location --p-method permanova --o-visualization beta_group_significance.qzv
```

## Evaluation signals

- beta_group_significance_results.tsv exists with pseudo-F and p-value fields populated for every row
- All reported p-values fall within [0, 1]
- Rows are present for both groupings (location and size fraction) across UniFrac metrics (unweighted and weighted)
- Separation verdicts are consistent with the p-values, with significance declared at p < 0.05
- unweighted_unifrac_emperor.qzv renders in QIIME 2 viewer and can be colored by both location and size fraction

## Limitations

- Results depend on the configured core_sampling_depth (500); changing rarefaction depth can alter UniFrac distances and PERMANOVA outcomes
- The paper demonstrates the workflow on a single 16S Lake Erie surface-water dataset; grouping factors and sample sizes are specific to that test run
- QIIME 2 visualizations are viewable as .qzv through QIIME 2 viewer or the HTML report and are not standalone figures without that viewing step

## Evidence

- [abstract] performs analyses of taxonomic, alpha, and beta diversity: "performs analyses of taxonomic, alpha, and beta diversity"
- [abstract] view qzv via viewer: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [methods] core metrics depth: [evidence span withheld — anchor status: not_found]
- [other] community separation finding: [evidence span withheld — anchor status: not_found]
- [readme] three workflow steps: "This script allows you to run one or more of the workflow steps at a time, specify specific config files, and set the maximum number of cores."
