---
name: otu-table-salinity-gradient-trend-analysis
description: Use when you have OTU tables for environmental samples ordered along
  a salinity gradient (e.g., river, lagoon, open sea 16S rRNA data) and need to test
  habitat differentiation via PERMANOVA and check the decreasing-OTU-with-salinity
  trend.
when_to_use_negative:
- The dataset has no environmental gradient metadata (e.g., a mock community study)
  — PERMANOVA habitat differentiation and OTU-vs-salinity trends are meaningless without
  habitat/salinity labels.
- You need OTU clustering via VSEARCH for the 12S rRNA marker gene — PEMA's table
  (and thus this downstream workflow as configured) does not support that combination.
- The input is a single-pipeline feature table with no cross-pipeline comparison intended
  — the PERMANOVA comparison step specifically requires PEMA, LotuS, and DADA2 outputs.
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3293
tools:
- name: PEMA
  role: Containerized pipeline (Docker/Singularity, BDS-implemented) that produces
    the OTU tables with taxonomies and abundances fed into the downstream ecological
    analysis
  repo: https://github.com/hariszaf/pema
- name: phyloseq
  role: R package invoked via PEMA's phyloseq_in_PEMA.R script to compute taxonomic
    composition, α- and β-diversity, and community matrices for PERMANOVA
  repo: http://joey711.github.io/phyloseq/index.html
- name: vegan
  role: R package whose functions (e.g., PERMANOVA) PEMA additionally performs for
    statistical comparisons between samples
  repo: https://cran.r-project.org/web/packages/vegan/index.html
- name: BDS
  role: BigDataScript language in which PEMA is implemented, providing checkpointing
    and intrinsic parallelization when generating the OTU tables
  repo: https://pcingola.github.io/BigDataScript/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:d4ad29f31f95e8cc600eaac6ea40548d2bcbaa522830fb67188b202779a462d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - LotuS
  - PEMA
  - QIIME 2
  - mothur
  - phyloseq
  - vegan
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# otu-table-salinity-gradient-trend-analysis

## Summary

Downstream ecological analysis of metabarcoding OTU tables (from PEMA, LotuS, DADA2, mothur, QIIME 2) along a river–lagoon–open sea salinity gradient, using the phyloseq and vegan R packages to compute community compositions, run PERMANOVA for habitat differentiation, and tabulate per-sample OTU numbers against salinity.

## When to use

Use when you have OTU tables for environmental samples ordered along a salinity gradient (e.g., river, lagoon, open sea 16S rRNA data) and need to test habitat differentiation via PERMANOVA and check the decreasing-OTU-with-salinity trend.

## When NOT to use

- The dataset has no environmental gradient metadata (e.g., a mock community study) — PERMANOVA habitat differentiation and OTU-vs-salinity trends are meaningless without habitat/salinity labels.
- You need OTU clustering via VSEARCH for the 12S rRNA marker gene — PEMA's table (and thus this downstream workflow as configured) does not support that combination.
- The input is a single-pipeline feature table with no cross-pipeline comparison intended — the PERMANOVA comparison step specifically requires PEMA, LotuS, and DADA2 outputs.

## Inputs

- OTU/ASV tables with taxonomies from PEMA (16S rRNA marker gene)
- OTU tables from LotuS, mothur, and QIIME 2 (DADA2 and Deblur variants) run on the same reads
- comma-separated metadata.csv with sample salinity and habitat (river/lagoon/open sea) labels
- Pavloudi et al. 16S rRNA .fastq read files from ENA (via ENA File Downloader)

## Outputs

- permanova_comparison.tsv with PERMANOVA statistics (F.Model, P) per pipeline
- taxonomic profiles and α-/β-diversity results per sample
- per-sample OTU counts tabulated against salinity

## How to apply

Generate the OTU tables by running PEMA (via Docker or Singularity with a parameters.tsv and mydata/ of .fastq.gz files) and compare against outputs of LotuS, mothur, and QIIME 2 (DADA2 and Deblur variants) run on the same reads. Analyze all pipeline OTU tables with the phyloseq R script provided with PEMA (phyloseq_in_PEMA.R plus a comma-separated metadata.csv), computing taxonomic profiles and α-/β-diversity. Run PERMANOVA on the community compositions of PEMA, LotuS, and DADA2 to test habitat differentiation among river, lagoon, and open sea samples. Tabulate per-sample OTU numbers against salinity to confirm the decreasing trend, writing statistics to permanova_comparison.tsv. The rationale is that phyloseq/vegan provide the statistical machinery PEMA exposes for downstream ecology, and cross-pipeline PERMANOVA separates pipeline effects from genuine environmental structure.

## Related tools

- **PEMA** (Containerized pipeline (Docker/Singularity, BDS-implemented) that produces the OTU tables with taxonomies and abundances fed into the downstream ecological analysis) — https://github.com/hariszaf/pema
- **phyloseq** (R package invoked via PEMA's phyloseq_in_PEMA.R script to compute taxonomic composition, α- and β-diversity, and community matrices for PERMANOVA) — http://joey711.github.io/phyloseq/index.html
- **vegan** (R package whose functions (e.g., PERMANOVA) PEMA additionally performs for statistical comparisons between samples) — https://cran.r-project.org/web/packages/vegan/index.html
- **BDS** (BigDataScript language in which PEMA is implemented, providing checkpointing and intrinsic parallelization when generating the OTU tables) — https://pcingola.github.io/BigDataScript/

## Examples

```
./pema_latest.bds  # inside the PEMA container with phyloseq_in_PEMA.R and metadata.csv in the mounted analysis directory; then permanova stats land in permanova_comparison.tsv
```

## Evaluation signals

- PERMANOVA on PEMA results must reproduce F.Model = 7.0718, P < 0.001; LotuS must give F.Model = 6.5901, P < 0.001
- Per-sample OTU numbers must decrease with increasing salinity across the river → lagoon → open sea samples
- permanova_comparison.tsv exists and contains F.Model and P values for PEMA, LotuS, and DADA2 community compositions
- metadata.csv is comma-separated and sample names in the phyloseq input match the OTU table sample IDs
- The phyloseq run completes only when phyloseq_in_PEMA.R keeps PEMA's mandatory main features intact while the user-customized sections are consistent with the OTU table

## Limitations

- PERMANOVA comparison was run only for PEMA, LotuS, and DADA2 results, not for all evaluated pipelines (mothur and QIIME 2 Deblur were compared on execution time and OTU counts).
- The parameters.tsv file version must exactly match the PEMA version used (e.g., pema:v.2.1.4 requires the v.2.1.4 parameters file), or the run will fail.
- PEMA names the analysis-directory files and directories strictly (parameters.tsv, mydata, phyloseq_in_PEMA.R, metadata.csv) and fails unless they are named exactly as described.

## Evidence

- [methods] PERMANOVA analysis for 16S rRNA marker gene data: "when the PERMANOVA analysis was run for the results of PEMA, LotuS, and DADA2"
- [methods] Alpha- and beta-diversity analysis scope: "This includes α- and β-diversity analysis, taxonomic composition, statistical comparisons, and calculation of correlations between samples"
- [intro] Phyloseq downstream analysis in PEMA: "In addition, ecological and phylogenetic analysis are facilitated via the “phyloseq” R package [27]."
- [readme] PEMA phyloseq and vegan functions: [evidence span withheld — anchor status: not_found]
- [readme] Required analysis-directory files for phyloseq runs: [evidence span withheld — anchor status: not_found]
