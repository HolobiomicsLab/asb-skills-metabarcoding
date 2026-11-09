---
name: microbiome-sample-metadata-stratification
description: Use when a denoised QIIME 2 feature table and metadata columns (e.g.,
  location, size fraction) are available and the goal is to determine if 16S communities
  are significantly distinguished by those grouping factors.
when_to_use_negative:
- The run artifacts are raw FASTQ reads rather than denoised QIIME 2 artifacts (.qza)
  — the qaqc/repseqs steps of Tourmaline must run first.
- Samples lack a categorical metadata column with at least two groups (e.g., no location
  or size-fraction labels), so PERMANOVA grouping is impossible.
- The goal is representative-sequence filtering (taxonomy, length, abundance, prevalence,
  or ID) rather than testing community separation by metadata groups.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: Runs core-metrics (UniFrac distance matrices, PCoA) and the diversity beta-group-significance
    (PERMANOVA) plugin used to test community separation by metadata group
  repo: https://github.com/aomlomics/tourmaline
- name: Tourmaline
  role: Snakemake-managed amplicon workflow that automates the QIIME 2 beta diversity
    analyses and exposes unfiltered run artifacts and configuration (e.g., plot_diversity,
    core_sampling_depth)
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2 viewer
  role: Web viewer for inspecting the .qzv Emperor ordinations and beta-group-significance
    plots and extracting pseudo-F and p-values
- name: Snakemake
  role: Workflow management system Tourmaline uses to orchestrate the beta diversity
    analysis steps
  repo: https://github.com/aomlomics/tourmaline
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:a381acbea900fe847e72b5bb4cc8ebfb58561af2bf47e84e6629990f387a983d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# microbiome-sample-metadata-stratification

## Summary

Tests whether microbiome communities (e.g., Lake Erie 16S rRNA amplicon samples) separate by sample metadata groups — such as sampling location (Open Water vs. Western Boundary) and size fraction (0.22-μm vs. 5.0-μm) — using UniFrac beta diversity, PCoA/Emperor ordination, and PERMANOVA in QIIME 2 within the Tourmaline workflow.

## When to use

Use when a denoised QIIME 2 feature table and metadata columns (e.g., location, size fraction) are available and the goal is to determine if 16S communities are significantly distinguished by those grouping factors.

## When NOT to use

- The run artifacts are raw FASTQ reads rather than denoised QIIME 2 artifacts (.qza) — the qaqc/repseqs steps of Tourmaline must run first.
- Samples lack a categorical metadata column with at least two groups (e.g., no location or size-fraction labels), so PERMANOVA grouping is impossible.
- The goal is representative-sequence filtering (taxonomy, length, abundance, prevalence, or ID) rather than testing community separation by metadata groups.

## Inputs

- Feature table artifact ([run_name]-table.qza) from the unfiltered Tourmaline run
- Representative sequence artifact ([run_name]-repseqs.qza)
- Sample metadata file mapping each sample to location (Open Water vs. Western Boundary) and size fraction (0.22-um vs. 5.0-um)

## Outputs

- unweighted_unifrac_emperor.qzv Emperor ordination visualization
- beta_group_significance.qzv PERMANOVA visualizations for unweighted and weighted UniFrac
- beta_group_significance_results.tsv summary table with grouping factor, distance metric, pseudo-F, p-value, and separation verdict

## How to apply

Load the unfiltered run artifacts ([run_name]-table.qza and [run_name]-repseqs.qza) into QIIME 2 together with the sample metadata file mapping each sample to location and size fraction. Compute UniFrac beta diversity distance matrices and PCoA ordinations with the QIIME 2 core-metrics workflow at core_sampling_depth: 500 (the depth configured when plot_diversity: true). Generate the Emperor ordination visualization (unweighted_unifrac_emperor.qzv) and inspect it colored by location and by size fraction. Run the QIIME 2 diversity beta-group-significance command (PERMANOVA) on the unweighted and weighted UniFrac distance matrices grouped by location and by size fraction, producing beta_group_significance.qzv. Extract the pseudo-F statistic and p-value from each visualization and record a separation verdict per grouping factor and metric (p < 0.05 indicates significant community separation).

## Related tools

- **QIIME 2** (Runs core-metrics (UniFrac distance matrices, PCoA) and the diversity beta-group-significance (PERMANOVA) plugin used to test community separation by metadata group) — https://github.com/aomlomics/tourmaline
- **Tourmaline** (Snakemake-managed amplicon workflow that automates the QIIME 2 beta diversity analyses and exposes unfiltered run artifacts and configuration (e.g., plot_diversity, core_sampling_depth)) — https://github.com/aomlomics/tourmaline
- **QIIME 2 viewer** (Web viewer for inspecting the .qzv Emperor ordinations and beta-group-significance plots and extracting pseudo-F and p-values)
- **Snakemake** (Workflow management system Tourmaline uses to orchestrate the beta diversity analysis steps) — https://github.com/aomlomics/tourmaline

## Examples

```
qiime diversity beta-group-significance --i-distance-matrix unweighted_unifrac_distance_matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column location --o-visualization beta_group_significance.qzv
```

## Evaluation signals

- beta_group_significance_results.tsv exists and contains pseudo-F and p-value fields for every row
- All reported p-values fall within [0, 1]
- The results table includes rows for both the location (Open Water vs. Western Boundary) and size-fraction (0.22-um vs. 5.0-um) groupings across UniFrac metrics
- unweighted_unifrac_emperor.qzv opens in QIIME 2 viewer and can be colored by both location and size fraction
- Separation verdicts are consistent with p < 0.05 recorded for each grouping factor / metric combination

## Limitations

- Results depend on the configured rarefaction depth (core_sampling_depth: 500); changing the sampling depth requires re-running and re-comparing runs.
- PERMANOVA significance for unfiltered runs may differ from filtered runs, since Tourmaline supports representative sequence filtering (taxonomy, length, abundance, prevalence, or ID) that can alter community distances.
- The paper demonstrates this on a single 16S rRNA amplicon dataset from Lake Erie surface water; other data types (e.g., 18S, ITS) or grouping factors may need adjusted parameters.

## Evidence

- [abstract] performs analyses of taxonomic, alpha, and beta diversity: "performs analyses of taxonomic, alpha, and beta diversity"
- [abstract] Lake Erie 16S dataset demonstration: [evidence span withheld — anchor status: not_found]
- [other] Beta diversity via core-metrics and PERMANOVA: [evidence span withheld — anchor status: not_found]
- [abstract] Interactive visualization through QIIME 2 viewer: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [readme] Tourmaline 2 wraps QIIME 2: "Tourmaline 2 is an amplicon sequence processing workflow for Illumina sequence data that uses [QIIME 2](https://qiime2.org) and the software packages it wraps."
