---
name: robbins-dominance-estimation
description: Use when computing Robbins or Chao1 on 16S amplicon feature tables and
  the denoiser choice (DADA2 vs DEBLUR) affects whether singleton counts exist for
  the metric.
when_to_use_negative:
- The feature table was produced with DADA2 (singletons removed) — Robbins and Chao1
  will be biased and must be computed on DEBLUR output instead.
- The analysis targets singleton-independent metrics (e.g. Observed features, Faith,
  entropy/dominance metrics without singleton dependence) — denoiser singleton treatment
  then need not constrain the choice.
- The data are not amplicon feature tables from QIIME 2's diversity module pipeline.
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: Suite in which samples are processed and whose diversity module computes Robbins
    and Chao1; workflows ran on QIIME 2022.2.0.
  repo: https://qiime2.org/
- name: DEBLUR
  role: Denoiser that keeps singletons; its output is the required baseline for Robbins/Chao1
    estimation.
- name: DADA2
  role: Contrast denoiser that removes singletons; used only to quantify the delta
    impact on singleton-dependent metrics.
- name: sra-tools
  role: Downloads each dataset from the SRA prior to QIIME 2 processing.
  repo: https://github.com/ncbi/sra-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:20857a92b86975d32eeb3b5fa7d81b76bcc8eb301e1b5a1f87c8df54768b7a15
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - QIIME 2
  - anaconda
  - sra-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# robbins-dominance-estimation

## Summary

Estimate singleton-dependent alpha diversity metrics (Robbins for dominance/unobserved estimation, Chao1 for unobserved microbe estimation) from amplicon feature tables denoised with a singleton-retaining tool (DEBLUR), since DADA2 discards singletons and renders these metrics invalid.

## When to use

Use when computing Robbins or Chao1 on 16S amplicon feature tables and the denoiser choice (DADA2 vs DEBLUR) affects whether singleton counts exist for the metric.

## When NOT to use

- The feature table was produced with DADA2 (singletons removed) — Robbins and Chao1 will be biased and must be computed on DEBLUR output instead.
- The analysis targets singleton-independent metrics (e.g. Observed features, Faith, entropy/dominance metrics without singleton dependence) — denoiser singleton treatment then need not constrain the choice.
- The data are not amplicon feature tables from QIIME 2's diversity module pipeline.

## Inputs

- SRA sequencing FASTQ datasets for human microbiome stool samples (accession numbers in ./datasets/xx_dataset_alias/SraRunTable.txt)
- DEBLUR-denoised feature table retaining singleton counts (QIIME 2 .qza artifact)
- Per-dataset sample metadata

## Outputs

- Alpha diversity metric vectors for Robbins and Chao1 from QIIME 2's diversity module
- datasets/dataset-name/alpha-out_dataset-name.tsv aggregated metric table
- alpha_aggregated.tsv combined metrics-plus-metadata table
- dada2_chao1_robbins_comparison.tsv delta table (per-sample and per-dataset DADA2 vs DEBLUR Chao1/Robbins differences)

## How to apply

Download raw sequencing data from SRA using sra-tools and process each dataset in QIIME 2 (the study used version 2022.2.0 on an anaconda environment) following QIIME2 best practices. Because DADA2 removes all singletons and Robbins and Chao1 require singleton counts, denoise the samples with DEBLUR (which keeps singletons) before running QIIME 2's diversity module on non-rarefied feature tables to preserve as much information as possible; rarefied tables can be used only for validation. Aggregate per-dataset metrics into datasets/dataset-name/alpha-out_dataset-name.tsv and combine with metadata. To quantify the denoiser's impact, compute per-sample and per-dataset deltas of Chao1 and Robbins between DADA2- and DEBLUR-processed runs and write them to dada2_chao1_robbins_comparison.tsv.

## Related tools

- **QIIME 2** (Suite in which samples are processed and whose diversity module computes Robbins and Chao1; workflows ran on QIIME 2022.2.0.) — https://qiime2.org/
- **DEBLUR** (Denoiser that keeps singletons; its output is the required baseline for Robbins/Chao1 estimation.)
- **DADA2** (Contrast denoiser that removes singletons; used only to quantify the delta impact on singleton-dependent metrics.)
- **sra-tools** (Downloads each dataset from the SRA prior to QIIME 2 processing.) — https://github.com/ncbi/sra-tools

## Examples

```
qiime diversity alpha --i-table deblur_table.qza --p-metric robbins --o-alpha-diversity robbins_vector.qza
```

## Evaluation signals

- Confirm the input feature table to the diversity run was DEBLUR-denoised and retains non-zero singleton counts, since DADA2 output removes them entirely.
- Verify the aggregated table exists at datasets/dataset-name/alpha-out_dataset-name.tsv and that alpha_aggregated.tsv joins metrics with each dataset's metadata.
- Check the delta analysis produces dada2_chao1_robbins_comparison.tsv with per-sample and per-dataset Chao1/Robbins deltas between DADA2 and DEBLUR runs.
- Confirm alpha diversity was computed on non-rarefied data, with rarefied-dataset results used only as validation.
- Inspect Robbins per-cohort values for subsampling artifacts (e.g. the KarenThai cohort pattern) before interpreting dominance estimates.

## Limitations

- Denoise tool must be reported as a source of variability when reporting estimated microorganism richness, since DADA2 and DEBLUR differ in singleton treatment.
- Robbins can exhibit subsampling artifacts at small sample sizes (hypothesized for the KarenThai cohort, requiring larger-sample resampling to verify).
- Version numbers for QIIME 2, DADA2 and DEBLUR were not reported for all processing steps, complicating exact reproduction.
- Raw sample data are not shared in the repository due to size; only accession numbers are provided.

## Evidence

- [discussion] DADA2 removes singletons and DEBLUR keeps them.: "DADA2 removes singletons and DEBLUR keeps them."
- [readme] All alpha diversity metrics were caluclated using QIIME2's diversity module.: "All alpha diversity metrics were caluclated using QIIME2's diversity module."
- [readme] Alpha metrics aggregated per dataset into a combined TSV.: "All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv"
- [discussion] Robbins subsampling artifact in the KarenThai cohort.: "According to Robbins (Fig. 4.B), there appears to be a subsampling artifact in the KarenThai cohort"
- [results] Singletons used in local polynomial regression against Faith.: "a local polynomial regression fitting was applied on the relationship between observed features (the selected metric proposed as representative for the Richness category"
