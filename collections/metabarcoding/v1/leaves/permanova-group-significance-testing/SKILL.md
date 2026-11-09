---
name: permanova-group-significance-testing
description: Use when you have QIIME 2 feature-table and rep-seqs artifacts plus sample
  metadata with categorical columns, and need to test whether beta diversity (unweighted/weighted
  UniFrac) differs between groups such as location or size fraction.
when_to_use_negative:
- The data are not yet denoised into a QIIME 2 feature table and rep-seqs artifact
  (e.g., still raw FASTQ needing the qaqc/DADA2 steps of Tourmaline).
- You need alpha diversity comparisons within samples rather than between-sample beta
  diversity group significance.
- Grouping columns are continuous rather than categorical, so PERMANOVA group separation
  is not the appropriate test.
edam_operation: http://edamontology.org/operation_3797
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: QIIME 2
  role: Runs the core-metrics workflow, UniFrac distance computation, Emperor ordination,
    and the diversity beta-group-significance (PERMANOVA) command
- name: Tourmaline
  role: Snakemake workflow that orchestrates the QIIME 2 beta diversity analyses and
    surfaces visualizations in the HTML report
  repo: github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system Tourmaline uses to manage commands, inputs, and
    outputs of the diversity steps
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2 viewer
  role: Web-viewable inspection of the .qzv ordination and PERMANOVA visualizations
    to read pseudo-F and p-values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:38c1e6eccd1035043c06b70d59c9869acde613f34823576d6f12fb8949351f88
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_3797
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# permanova-group-significance-testing

## Summary

Tests whether microbial communities separate between metadata groupings (e.g., sampling location and size fraction) by running QIIME 2 PERMANOVA (beta-group-significance) on UniFrac distance matrices and extracting pseudo-F and p-values, complemented by Emperor ordination inspection.

## When to use

Use when you have QIIME 2 feature-table and rep-seqs artifacts plus sample metadata with categorical columns, and need to test whether beta diversity (unweighted/weighted UniFrac) differs between groups such as location or size fraction.

## When NOT to use

- The data are not yet denoised into a QIIME 2 feature table and rep-seqs artifact (e.g., still raw FASTQ needing the qaqc/DADA2 steps of Tourmaline).
- You need alpha diversity comparisons within samples rather than between-sample beta diversity group significance.
- Grouping columns are continuous rather than categorical, so PERMANOVA group separation is not the appropriate test.

## Inputs

- [run_name]-table.qza (QIIME 2 feature table artifact)
- [run_name]-repseqs.qza (QIIME 2 representative sequences artifact)
- Sample metadata file with categorical columns (e.g., location: Open Water vs. Western Boundary; size fraction: 0.22-um vs. 5.0-um)
- core_sampling_depth setting (500 in the Lake Erie test run)

## Outputs

- unweighted_unifrac_emperor.qzv (Emperor PCoA ordination visualization)
- beta_group_significance.qzv (PERMANOVA test visualizations for each grouping factor and metric)
- beta_group_significance_results.tsv (table of grouping factor, distance metric, pseudo-F, p-value, and separation verdict)
- UniFrac distance matrices and PCoA ordinations from the core-metrics workflow

## How to apply

Load the run's [run_name]-table.qza and [run_name]-repseqs.qza into QIIME 2 together with the sample metadata mapping each sample to groupings such as location (Open Water vs. Western Boundary) and size fraction (0.22-um vs. 5.0-um). Compute UniFrac distance matrices and PCoA ordinations with the QIIME 2 core-metrics workflow at core_sampling_depth: 500 (the depth configured when plot_diversity: true), producing the unweighted_unifrac_emperor.qzv ordination. Inspect the Emperor plot colored by each grouping factor, then run the QIIME 2 diversity beta-group-significance command (PERMANOVA) on the unweighted and weighted UniFrac distance matrices grouped by location and by size fraction, yielding beta_group_significance.qzv. Extract the pseudo-F statistic and p-value from each .qzv, record a separation verdict for each grouping factor using the p < 0.05 threshold, and write a summary table (grouping factor, distance metric, pseudo-F, p-value, verdict) to beta_group_significance_results.tsv. In the Tourmaline context this corresponds to the beta diversity step of the workflow, whose visualizations are also surfaced via the HTML report and QIIME 2 viewer.

## Related tools

- **QIIME 2** (Runs the core-metrics workflow, UniFrac distance computation, Emperor ordination, and the diversity beta-group-significance (PERMANOVA) command)
- **Tourmaline** (Snakemake workflow that orchestrates the QIIME 2 beta diversity analyses and surfaces visualizations in the HTML report) — github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system Tourmaline uses to manage commands, inputs, and outputs of the diversity steps) — github.com/aomlomics/tourmaline
- **QIIME 2 viewer** (Web-viewable inspection of the .qzv ordination and PERMANOVA visualizations to read pseudo-F and p-values)

## Examples

```
qiime diversity beta-group-significance --i-distance-matrix unweighted_unifrac_distance_matrix.qza --m-metadata-file sample-metadata.tsv --m-metadata-column location --p-method permanova --o-visualization beta_group_significance.qzv
```

## Evaluation signals

- beta_group_significance_results.tsv exists with pseudo-F and p-value fields populated for every row
- All reported p-values fall within [0, 1]
- Rows are present for both the location (Open Water vs. Western Boundary) and size-fraction (0.22-um vs. 5.0-um) groupings across unweighted and weighted UniFrac metrics
- Separation verdicts are consistent with the p < 0.05 threshold applied to each row
- The unweighted_unifrac_emperor.qzv renders in QIIME 2 viewer and supports coloring by location and by size fraction

## Limitations

- PERMANOVA results depend on the rarefaction depth; the Lake Erie test run used core_sampling_depth: 500, and changing this depth changes the distance matrices and test outcomes.
- Significant group separation (p < 0.05) indicates communities are distinguished, but does not by itself identify which taxa drive the separation.
- The pseudo-F and p-values must be read from the .qzv visualizations via QIIME 2 viewer; they are not emitted as plain-text statistics by the workflow.

## Evidence

- [abstract] performs analyses of taxonomic, alpha, and beta diversity: "performs analyses of taxonomic, alpha, and beta diversity"
- [other] core-metrics at configured depth: [evidence span withheld — anchor status: not_found]
- [abstract] view interactive visualizations: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [abstract] built using Snakemake: "We developed Tourmaline, a Python-based workflow that implements QIIME 2 and is built using the Snakemake workflow management system"
