---
name: singleton-sensitivity-analysis-of-alpha-metrics
description: Use when estimating singleton-dependent richness metrics (Chao1, Robbins)
  and you must decide whether DADA2-denoised data can support them, or when comparing
  denoiser choices as a source of variability in estimated microbial richness.
when_to_use_negative:
- Metrics that do not depend on singleton counts (e.g. Observed features, Faith's
  PD, entropy/dominance metrics) — singleton removal is not the relevant sensitivity
  axis
- The feature table you hold is already from a single denoiser and the question is
  metric choice, not denoiser choice — use the paper's metric-category guidelines
  instead
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME2
  role: Suite in which both DADA2 and DEBLUR denoising runs and the diversity-module
    alpha metric computation are executed
  repo: https://qiime2.org/
- name: DADA2
  role: Denoiser whose singleton-removal behavior is the perturbation being tested
    against Chao1 and Robbins
- name: DEBLUR
  role: Denoiser that retains singletons, providing the baseline feature table for
    singleton-dependent metrics
- name: sra-tools
  role: Downloads the SRA sequencing datasets used in the paired DADA2/DEBLUR comparison
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
  source_content_hash: sha256:56dc2de057d5b1d90c59f9c2735ae22925602bf46b19c572f6538b09ec89d10a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - DEBLUR
  - QIIME2
  - anaconda
  - sra-tools
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# singleton-sensitivity-analysis-of-alpha-metrics

## Summary

Quantifies how denoiser-specific singleton handling distorts singleton-dependent alpha diversity metrics (Chao1, Robbins) by recomputing these metrics on feature tables produced with DADA2 (which discards singletons) versus DEBLUR (which retains them) in QIIME2 2022.2.0.

## When to use

Use when estimating singleton-dependent richness metrics (Chao1, Robbins) and you must decide whether DADA2-denoised data can support them, or when comparing denoiser choices as a source of variability in estimated microbial richness.

## When NOT to use

- Metrics that do not depend on singleton counts (e.g. Observed features, Faith's PD, entropy/dominance metrics) — singleton removal is not the relevant sensitivity axis
- The feature table you hold is already from a single denoiser and the question is metric choice, not denoiser choice — use the paper's metric-category guidelines instead

## Inputs

- SRA sequencing datasets (accession numbers e.g. SraRunTable.txt per dataset folder)
- DADA2-denoised QIIME2 feature table (singletons removed)
- DEBLUR-denoised QIIME2 feature table (singletons retained)
- Sample metadata for each dataset

## Outputs

- Per-sample and per-dataset deltas of Chao1 and Robbins between DADA2 and DEBLUR runs (e.g. dada2_chao1_robbins_comparison.tsv)
- Aggregated alpha diversity tables (alpha-out_dataset-name.tsv) with metrics joined to metadata (alpha_aggregated.tsv)

## How to apply

Process each SRA dataset twice in QIIME2 (2022.2.0) following QIIME2 best practices: once with DADA2 and once with DEBLUR, since DADA2 removes singletons while DEBLUR keeps them. Compute Chao1 and Robbins with QIIME2's diversity module on non-rarefied feature tables to preserve as much information as possible, aggregating per-dataset metrics into datasets/dataset-name/alpha-out_dataset-name.tsv and combining with metadata. Treat the DEBLUR (singletons-retained) table as the baseline, then compute per-sample and per-dataset deltas of Chao1 and Robbins between the DADA2 and DEBLUR runs to quantify the impact of singleton removal. Report the denoise tool alongside any estimated richness, because the singleton-treatment difference is itself a source of variability.

## Related tools

- **QIIME2** (Suite in which both DADA2 and DEBLUR denoising runs and the diversity-module alpha metric computation are executed) — https://qiime2.org/
- **DADA2** (Denoiser whose singleton-removal behavior is the perturbation being tested against Chao1 and Robbins)
- **DEBLUR** (Denoiser that retains singletons, providing the baseline feature table for singleton-dependent metrics)
- **sra-tools** (Downloads the SRA sequencing datasets used in the paired DADA2/DEBLUR comparison) — https://github.com/ncbi/sra-tools
- **anaconda** (Virtual environment hosting the QIIME2 installation used for all sample processing)

## Examples

```
source venv/bin/activate && pip install -r requirements.txt  # then run notebooks computing Chao1/Robbins deltas between DADA2 and DEBLUR alpha tables
```

## Evaluation signals

- Chao1 and Robbins computed on the DADA2 table should be systematically deflated relative to the DEBLUR baseline, since DADA2 removes all singletons
- Per-dataset delta tables (dada2_chao1_robbins_comparison.tsv) contain finite deltas for every sample in every dataset
- Both DADA2 and DEBLUR runs cover the same sample set processed via QIIME2 2022.2.0 best practices after per-dataset QC (read quality, adapter trimming)
- All alpha metrics were computed on non-rarefied feature tables, with results validated against rarefied datasets
- Aggregate table alpha_aggregated.tsv joins each sample's metrics to its dataset metadata without orphan rows

## Limitations

- Raw sample data is not shared in the repository due to size; only accession numbers are provided per dataset folder, so full reproduction requires re-downloading from SRA
- No version numbers are reported for DADA2 or DEBLUR themselves, so singleton-treatment behavior may differ across denoiser versions
- The denoise tool must be reported when estimating richness; unreported denoiser choice is itself a source of variability in Chao1/Robbins values

## Evidence

- [discussion] DADA2 removes singletons and DEBLUR keeps them.: "DADA2 removes singletons and DEBLUR keeps them."
- [discussion] Denoisers differ in singleton treatment: "DADA223 and DEBLUR24 not only have different algorithmic denoising techniques but also have relevant differences in their singletons treatment approach."
- [discussion] Denoiser as source of variability: "This difference should be considered as a source of variability when reporting the estimated microorganism richness of the sample, if the denoise tool is not reported."
- [results] Both denoisers applied to all samples: "all samples were processed using DADA2 and DEBLUR"
- [readme] Alpha metrics via QIIME2 diversity module: "All alpha diversity metrics were caluclated using QIIME2's diversity module. All metrics for each dataset were aggreted on a table called datasets/dataset-name/alpha-out_dataset-name.tsv."
- [readme] QIIME2 version used: "Our workflows were ran on qiime 2022.2.0, but should be compatible with newer versions of the code."
