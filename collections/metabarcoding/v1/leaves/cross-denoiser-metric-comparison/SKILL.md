---
name: cross-denoiser-metric-comparison
description: Use when decision support for choosing or reporting a denoising tool
  when downstream alpha diversity metrics depend on singleton counts (e.g. Chao1,
  Robbins), or when estimating the bias introduced by singleton removal.
when_to_use_negative:
- The downstream analysis uses singleton-independent metrics (e.g. observed features,
  Shannon, Faith) where DADA2-vs-DEBLUR differences are not the primary concern
- You already have a single denoised feature table and no paired DADA2/DEBLUR runs
  to compare — there is no delta to compute
- Input is rarefied data; the article calculated alpha diversity metrics on non-rarefied
  data to preserve as much information as possible (rarefied runs were only used for
  validation)
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: QIIME2 2022.2.0
  role: Pipeline framework in which both denoising runs and alpha diversity computation
    via the diversity module are performed
  repo: https://qiime2.org/
- name: DADA2
  role: Denoiser that removes singletons, used to produce the singleton-free feature
    table for the comparison
- name: DEBLUR
  role: Denoiser that keeps singletons; its output is the baseline for singleton-dependent
    metrics
- name: sra-tools
  role: Downloads the raw sequencing datasets from the SRA
  repo: https://github.com/ncbi/sra-tools
- name: anaconda
  role: Virtual environment hosting the QIIME2 installation used for all sample processing
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-024-77864-y
    title: Key features and guidelines for the application of microbial alpha diversity
      metrics
  source_content_hash: sha256:c8a57b55f7b95ca0f25c149732bd7d41f3df0eab1936e9ded0f25a1ae1608372
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - QIIME2 2022.2.0
  - anaconda
  - sra-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# cross-denoiser-metric-comparison

## Summary

Quantifies how denoiser choice (DADA2 vs DEBLUR, which differ in singleton treatment) biases singleton-dependent alpha diversity metrics such as Chao1 and Robbins, by computing per-sample deltas between DADA2- and DEBLUR-derived feature tables.

## When to use

Decision support for choosing or reporting a denoising tool when downstream alpha diversity metrics depend on singleton counts (e.g. Chao1, Robbins), or when estimating the bias introduced by singleton removal.

## When NOT to use

- The downstream analysis uses singleton-independent metrics (e.g. observed features, Shannon, Faith) where DADA2-vs-DEBLUR differences are not the primary concern
- You already have a single denoised feature table and no paired DADA2/DEBLUR runs to compare — there is no delta to compute
- Input is rarefied data; the article calculated alpha diversity metrics on non-rarefied data to preserve as much information as possible (rarefied runs were only used for validation)

## Inputs

- SRA sequencing datasets (FASTQ, accession numbers per dataset folder, e.g. SraRunTable.txt)
- DADA2-denoised QIIME2 feature table (singletons removed)
- DEBLUR-denoised QIIME2 feature table (singletons retained)
- Per-sample metadata for each of the 13 human microbiome projects (4,596 stool samples)

## Outputs

- datasets/dataset-name/alpha-out_dataset-name.tsv: aggregated Chao1 and Robbins metrics per dataset
- alpha_aggregated.tsv: all alpha diversity metrics combined with sample metadata
- dada2_chao1_robbins_comparison.tsv: per-sample and per-dataset DADA2-vs-DEBLUR deltas for Chao1 and Robbins

## How to apply

Download the SRA datasets with sra-tools and process every sample twice inside QIIME2 2022.2.0 following QIIME2 best practices: once with DADA2 (which removes singletons) and once with chained DEBLUR (which retains them). Compute alpha diversity metrics — specifically Chao1 and Robbins — with QIIME2's diversity module on non-rarefied feature tables, aggregating each dataset's metrics into datasets/dataset-name/alpha-out_dataset-name.tsv and combining them with sample metadata. Then compute per-sample and per-dataset deltas of the singleton-dependent metrics between the DADA2 and DEBLUR runs and write the comparison to a table such as dada2_chao1_robbins_comparison.tsv. The rationale is that metrics requiring singleton counts (Chao1 estimates unobserved richness; Robbins measures sampling heterogeneity) are structurally biased when computed on DADA2 output, so the study's downstream alpha diversity analyses were performed on DEBLUR data.

## Related tools

- **QIIME2 2022.2.0** (Pipeline framework in which both denoising runs and alpha diversity computation via the diversity module are performed) — https://qiime2.org/
- **DADA2** (Denoiser that removes singletons, used to produce the singleton-free feature table for the comparison)
- **DEBLUR** (Denoiser that keeps singletons; its output is the baseline for singleton-dependent metrics)
- **sra-tools** (Downloads the raw sequencing datasets from the SRA) — https://github.com/ncbi/sra-tools
- **anaconda** (Virtual environment hosting the QIIME2 installation used for all sample processing)

## Examples

```
qiime diversity alpha --i-table deblur_table.qza --p-metric chao1 --o-alpha-diversity chao1_deblur.qza
```

## Evaluation signals

- The DEBLUR feature table contains non-zero singleton counts for samples where the DADA2 table has none (singleton removal confirmed for DADA2)
- Chao1 and Robbins values computed from DADA2 tables show systematic negative deltas relative to DEBLUR values in dada2_chao1_robbins_comparison.tsv
- Each dataset's alpha metrics aggregate into datasets/dataset-name/alpha-out_dataset-name.tsv and merge cleanly with the dataset's metadata (sample IDs match)
- Metric computation matches QIIME2's diversity module output and was run on non-rarefied tables (rarefied runs used only as validation)
- All 13 datasets (4,596 stool samples) processed through both DADA2 and DEBLUR pipelines without dropped projects other than documented QC exclusions

## Limitations

- Denoise tool must be reported as a source of variability for estimated richness; unreported denoiser choice makes Chao1/Robbins values incomparable across studies
- Raw sample data are not shared in the repository due to size — only accession numbers in each dataset's folder (./datasets/xx_dataset_alias/SraRunTable.txt)
- Paired-end samples that only had a single read were removed, and projects lacking associated metadata were excluded
- No version numbers reported for DADA2 or DEBLUR in the discussion, limiting exact reproducibility of the denoising step

## Evidence

- [discussion] Singleton treatment difference: "DADA2 removes singletons and DEBLUR keeps them."
- [results] Both denoisers applied to all samples: "all samples were processed using DADA2 and DEBLUR"
- [discussion] Singleton treatment as algorithmic difference: "DADA223 and DEBLUR24 not only have different algorithmic denoising techniques but also have relevant differences in their singletons treatment approach."
- [results] Non-rarefied data rationale: "alpha diversity metrics were calculated with non-rarefied data to preserve as much information as possible when calculating them."
- [readme] Metrics computed with QIIME2 diversity module: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
- [discussion] Denoiser reporting as variability source: "This difference should be considered as a source of variability when reporting the estimated microorganism richness of the sample, if the denoise tool is not reported."
