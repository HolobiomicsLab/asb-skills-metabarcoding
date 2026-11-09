---
name: alpha-beta-diversity-analysis
description: Use when a denoised DADA2/Deblur feature table (.qza) and sample metadata
  are available and the goal is to test whether amplicon communities separate by metadata
  factors such as location or size fraction.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been denoised into a feature table by
  DADA2 or Deblur
- Representative sequences have not yet had taxonomy assigned or the feature table
  has not been filtered when the filtered run's diversity outputs are the target
- The analysis goal is denoising, trimming, or taxonomy classification rather than
  community separation testing
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: Executes the diversity analyses (core-metrics, beta-group-significance) on
    the feature table and repseqs artifacts
- name: Tourmaline
  role: Snakemake workflow that wraps the QIIME 2 diversity commands as rules such
    as dada2_pe_diversity_unfiltered
  repo: https://github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system that runs the diversity rules defined in the Tourmaline
    Snakefile
- name: DADA2
  role: Produces the denoised feature table and representative sequences that serve
    as diversity inputs
  repo: https://github.com/benjjneb/dada2
- name: QIIME 2 viewer
  role: Used to inspect the Emperor ordination and beta-group-significance .qzv visualizations
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:6a8bc864bd8c83432334ec9572fb80ba7e882398eb21003ed9abdd58f6a89de8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - DADA2
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# alpha-beta-diversity-analysis

## Summary

Computes taxonomic, alpha, and beta diversity analyses on a denoised feature table using QIIME 2's core-metrics workflow within the Tourmaline Snakemake pipeline, including UniFrac distance matrices, PCoA ordinations, and PERMANOVA group-significance tests.

## When to use

Use when a denoised DADA2/Deblur feature table (.qza) and sample metadata are available and the goal is to test whether amplicon communities separate by metadata factors such as location or size fraction.

## When NOT to use

- Input is raw FASTQ reads that have not yet been denoised into a feature table by DADA2 or Deblur
- Representative sequences have not yet had taxonomy assigned or the feature table has not been filtered when the filtered run's diversity outputs are the target
- The analysis goal is denoising, trimming, or taxonomy classification rather than community separation testing

## Inputs

- Denoised feature table artifact ([run_name]-table.qza)
- Representative sequences artifact ([run_name]-repseqs.qza)
- Sample metadata file mapping samples to grouping factors (e.g., location, size fraction)
- Configured core sampling depth (e.g., core_sampling_depth: 500)

## Outputs

- UniFrac (weighted and unweighted) distance matrices and PCoA ordinations
- Emperor ordination visualization (unweighted_unifrac_emperor.qzv)
- Beta group significance (PERMANOVA) visualization (beta_group_significance.qzv) with pseudo-F and p-values
- Summary table of grouping factor, distance metric, pseudo-F, p-value, and separation verdict (e.g., beta_group_significance_results.tsv)
- Alpha diversity and taxonomic diversity outputs from the core-metrics workflow

## How to apply

Run the Tourmaline rule dada2_pe_diversity_unfiltered (or the q2-diversity core-metrics step) on the run's feature table and representative sequences ([run_name]-table.qza and [run_name]-repseqs.qza) together with the sample metadata file. Set core_sampling_depth to the configured rarefaction depth (500 for the bundled Lake Erie 16S test dataset when plot_diversity: true) to compute UniFrac distance matrices and PCoA ordinations. Generate the unweighted_unifrac_emperor.qzv ordination and inspect it colored by each metadata factor (e.g., Open Water vs. Western Boundary; 0.22-um vs. 5.0-um). Run QIIME 2 diversity beta-group-significance (PERMANOVA) on the unweighted and weighted UniFrac distance matrices grouped by each factor, extract pseudo-F and p-values from the resulting beta_group_significance.qzv files, and record separation as significant when p < 0.05.

## Related tools

- **QIIME 2** (Executes the diversity analyses (core-metrics, beta-group-significance) on the feature table and repseqs artifacts)
- **Tourmaline** (Snakemake workflow that wraps the QIIME 2 diversity commands as rules such as dada2_pe_diversity_unfiltered) — https://github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system that runs the diversity rules defined in the Tourmaline Snakefile)
- **DADA2** (Produces the denoised feature table and representative sequences that serve as diversity inputs) — https://github.com/benjjneb/dada2
- **QIIME 2 viewer** (Used to inspect the Emperor ordination and beta-group-significance .qzv visualizations)

## Examples

```
snakemake --snakefile Snakefile dada2_pe_diversity_unfiltered --cores N
```

## Evaluation signals

- The results table (e.g., beta_group_significance_results.tsv) exists with pseudo-F and p-value fields present
- All p-values fall within [0, 1]
- Rows are reported for both the location (Open Water vs. Western Boundary) and size-fraction (0.22-um vs. 5.0-um) groupings across UniFrac metrics
- The Emperor visualization (unweighted_unifrac_emperor.qzv) can be opened in QIIME 2 viewer and colored by each metadata factor
- Community separation is reported as significant only where p < 0.05 for the corresponding UniFrac distance matrix

## Limitations

- Results depend on the configured core_sampling_depth (500 for the Lake Erie test run); a different depth changes rarefied distance matrices
- Separation verdicts rely on PERMANOVA significance (p < 0.05) on unweighted and weighted UniFrac; factors with non-significant p-values should not be reported as separating communities
- The unfiltered run's diversity outputs use the unfiltered feature table; filtered repseq runs require their corresponding diversity rule

## Evidence

- [other] Beta diversity analysis of the Lake Erie test data shows 16S communities are distinguished both by location (Open Water vs. Western Boundary) and by size fraction (0.22-μm vs. 5.0-μm).: [evidence span withheld — anchor status: not_found]
- [other] Compute UniFrac beta diversity distance matrices and PCoA ordinations with the QIIME 2 core-metrics workflow at core_sampling_depth: 500: [evidence span withheld — anchor status: not_found]
- [abstract] performs analyses of taxonomic, alpha, and beta diversity: [evidence span withheld — anchor status: not_found]
- [intro] Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files: "Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files"
