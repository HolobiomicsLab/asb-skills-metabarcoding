---
name: unifrac-ordination-visualization
description: Use when a denoised QIIME 2 feature table (.qza) and representative sequences
  (.qza) are available together with sample metadata, and the goal is to test or visualize
  community separation between metadata-defined groups via UniFrac ordination and
  PERMANOVA.
when_to_use_negative:
- The feature table and representative sequences have not yet been produced by denoising
  (no .qza artifacts) — UniFrac requires aligned representative sequences, so run
  qaqc/repseqs steps first.
- The goal is taxonomic assignment or alpha diversity summarization rather than between-sample
  (beta) diversity ordination and group-separation testing.
- The metadata file lacks the grouping columns (e.g., location or size fraction) needed
  for beta-group-significance PERMANOVA tests.
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: QIIME 2
  role: Runs the core-metrics workflow and the diversity beta-group-significance (PERMANOVA)
    command to produce UniFrac distance matrices, PCoA ordinations, and Emperor visualizations
  repo: https://qiime2.org
- name: Tourmaline
  role: 'Snakemake-managed workflow that orchestrates the QIIME 2 beta diversity step
    (configured via plot_diversity: true and core_sampling_depth: 500) within the
    amplicon pipeline'
  repo: https://github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system used by Tourmaline to manage commands, inputs,
    and outputs for the diversity steps
  repo: https://snakemake.readthedocs.io/en/stable/
- name: QIIME 2 viewer
  role: Web-viewable interface for inspecting the unweighted_unifrac_emperor.qzv and
    beta_group_significance.qzv visualizations
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:04b3307a6e12e91cf14db734409eb2d7301e74ebfb08d556acfaeda1a0af6589
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
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# unifrac-ordination-visualization

## Summary

Compute UniFrac beta-diversity distance matrices, PCoA ordinations, and PERMANOVA beta-group-significance tests with QIIME 2 to test whether 16S amplicon communities separate by sample metadata groupings (e.g., location: Open Water vs. Western Boundary; size fraction: 0.22-um vs. 5.0-um), and visualize the ordination interactively as an Emperor plot viewable through QIIME 2 viewer.

## When to use

Use when a denoised QIIME 2 feature table (.qza) and representative sequences (.qza) are available together with sample metadata, and the goal is to test or visualize community separation between metadata-defined groups via UniFrac ordination and PERMANOVA.

## When NOT to use

- The feature table and representative sequences have not yet been produced by denoising (no .qza artifacts) — UniFrac requires aligned representative sequences, so run qaqc/repseqs steps first.
- The goal is taxonomic assignment or alpha diversity summarization rather than between-sample (beta) diversity ordination and group-separation testing.
- The metadata file lacks the grouping columns (e.g., location or size fraction) needed for beta-group-significance PERMANOVA tests.

## Inputs

- Feature table artifact ([run_name]-table.qza) from the denoised (DADA2 or Deblur) run
- Representative sequences artifact ([run_name]-repseqs.qza)
- Sample metadata file mapping samples to grouping columns (e.g., location: Open Water vs. Western Boundary; size fraction: 0.22-um vs. 5.0-um)

## Outputs

- Unweighted and weighted UniFrac distance matrices (.qza)
- PCoA ordination artifacts (.qza) computed at core sampling depth 500
- Emperor ordination visualization (unweighted_unifrac_emperor.qzv)
- beta_group_significance.qzv containing pseudo-F and p-value per grouping factor and metric
- Summary table beta_group_significance_results.tsv (grouping factor, distance metric, pseudo-F, p-value, separation verdict)

## How to apply

Load the run's [run_name]-table.qza and [run_name]-repseqs.qza artifacts plus a sample metadata file mapping each sample to grouping factors such as location (Open Water vs. Western Boundary) and size fraction (0.22-um vs. 5.0-um). Run the QIIME 2 core-metrics workflow at core_sampling_depth 500 (the depth configured when plot_diversity: true in Tourmaline) to compute unweighted and weighted UniFrac distance matrices and PCoA ordinations, then generate the Emperor ordination visualization (unweighted_unifrac_emperor.qzv) and inspect it colored by each grouping factor. For each factor, run the QIIME 2 diversity beta-group-significance command with the PERMANOVA method on the unweighted and weighted UniFrac matrices to produce beta_group_significance.qzv, extract the pseudo-F statistic and p-value, and declare significant separation when p < 0.05. Record results as a summary table (grouping factor, distance metric, pseudo-F, p-value, separation verdict), e.g. to beta_group_significance_results.tsv.

## Related tools

- **QIIME 2** (Runs the core-metrics workflow and the diversity beta-group-significance (PERMANOVA) command to produce UniFrac distance matrices, PCoA ordinations, and Emperor visualizations) — https://qiime2.org
- **Tourmaline** (Snakemake-managed workflow that orchestrates the QIIME 2 beta diversity step (configured via plot_diversity: true and core_sampling_depth: 500) within the amplicon pipeline) — https://github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system used by Tourmaline to manage commands, inputs, and outputs for the diversity steps) — https://snakemake.readthedocs.io/en/stable/
- **QIIME 2 viewer** (Web-viewable interface for inspecting the unweighted_unifrac_emperor.qzv and beta_group_significance.qzv visualizations)

## Examples

```
qiime diversity core-metrics --i-table table.qza --p-sampling-depth 500 --m-metadata-file metadata.tsv --output-dir core_metrics
```

## Evaluation signals

- beta_group_significance_results.tsv exists with both pseudo-F and p-value fields populated for rows covering both the location and size-fraction groupings across UniFrac metrics.
- All reported p-values fall within [0, 1].
- unweighted_unifrac_emperor.qzv is produced and renders correctly in QIIME 2 viewer when colored by location and by size fraction.
- Ordination was computed at the configured core sampling depth of 500, matching the plot_diversity: true configuration.
- Each grouping factor with p < 0.05 is recorded as showing significant community separation (per the PERMANOVA verdict).

## Limitations

- Samples with fewer reads than the core sampling depth (500) are dropped from the core-metrics ordination, which can change group composition between tests.
- Separation verdicts depend on the metadata file correctly labeling every sample; missing or inconsistent location/size-fraction entries will invalidate the PERMANOVA test.
- Pseudo-F statistics and p-values must be extracted manually from the beta_group_significance.qzv (e.g., via QIIME 2 viewer) before writing the results table.

## Evidence

- [abstract] performs analyses of taxonomic, alpha, and beta diversity: "performs analyses of taxonomic, alpha, and beta diversity"
- [intro] an amplicon analysis pipeline that uses Snakemake to run QIIME 2 commands for core analysis and interactive visualization: "an amplicon analysis pipeline that uses Snakemake to run QIIME 2 commands for core analysis and interactive visualization"
- [readme] Tourmaline 2 manages commands, inputs, and outputs using the [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow management system.: "Tourmaline 2 manages commands, inputs, and outputs using the [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow management system."
