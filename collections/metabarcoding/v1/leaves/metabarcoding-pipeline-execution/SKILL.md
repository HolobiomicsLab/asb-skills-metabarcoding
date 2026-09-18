---
name: metabarcoding-pipeline-execution
description: Use when raw amplicon FASTQ datasets with mock/negative controls and
  replicates must be processed through multiple metabarcoding pipelines (VTAM, DADA2+LULU,
  OBITools3+metabaR) under one reproducible setup for fair benchmarking.
when_to_use_negative:
- Input is already a curated ASV/feature table rather than raw paired-end FASTQ with
  control samples — the skill's filtering parametrization requires controls and replicates
- Study lacks mock community and negative control samples, since VTAM's filter optimization
  explicitly processes negative control and mock samples to set thresholds
- The goal is a single-pipeline production analysis rather than a controlled cross-pipeline
  comparison under identical hardware and reference databases
edam_operation: http://edamontology.org/operation_3798
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: VTAM
  role: 'Primary pipeline: merge, sortreads, filter (FilterLNF, FilterPCRerror, FilterMinReplicateNumber,
    FilterRenkonen, FilterChimera, FilterCodonStop, FilterIndel), optimize, and taxassign
    commands'
  repo: https://github.com/aitgon/vtam
- name: DADA2
  role: Denoising engine of the DALU comparison pipeline (v1.12.1) with marker-specific
    truncation lengths
- name: LULU
  role: Post-denoising curation of DALU ASVs (v0.1.0)
- name: OBITools3
  role: Denoising (obi uniq/grep/clean) engine of the OBIbaR comparison pipeline (v3.0.1b19)
- name: metabaR
  role: Occurrence filtering and replicate pooling in the OBIbaR pipeline (v1.0.0)
- name: cutadapt
  role: Primer/tag trimming in the preparation workflow
- name: vsearch
  role: VTAM dependency for sequence processing (e.g. chimera handling)
- name: Snakemake
  role: Workflow orchestration of prep, analysis, and min_readcount cut-off sweep
    Snakemake files
- name: Singularity
  role: Containerizes the whole benchmark via vtam_benchmark.def for reproducibility
- name: Python
  role: VTAM implementation language and plotting scripts (scripts/plt_sensitivity_precision.py)
  repo: https://github.com/aitgon/vtam
merged_aliases:
- metabarcoding-pipeline-benchmarking
merged_alias_records:
- alias: metabarcoding-pipeline-benchmarking
  slug: metabarcoding-pipeline-benchmarking
  cosine_score: 0.8646
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:e152fe39bf4d34b004bf11805f9f2d6fa71bc923897e4dfcb7265cdca4206b22
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aitgon/vtam_benchmark
  repo_url: https://github.com/aitgon/vtam
  tools:
  - DADA2
  - LULU
  - OBITools3
  - Python
  - R
  - Singularity
  - Snakemake
  - VTAM
  - cutadapt
  - metabaR
  - vsearch
  edam_operation: http://edamontology.org/operation_3798
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8823529411764706
n_claims: 17
---

# metabarcoding-pipeline-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run and benchmark the VTAM, DALU, and OBIbaR metabarcoding pipelines end-to-end under identical Snakemake/Singularity conditions on bat, fish, and shark datasets, using control-sample-based filtering to compare sensitivity, precision, runtime, and memory.

## When to use

Use when raw amplicon FASTQ datasets with mock/negative controls and replicates must be processed through multiple metabarcoding pipelines (VTAM, DADA2+LULU, OBITools3+metabaR) under one reproducible setup for fair benchmarking.

## When NOT to use

- Input is already a curated ASV/feature table rather than raw paired-end FASTQ with control samples — the skill's filtering parametrization requires controls and replicates
- Study lacks mock community and negative control samples, since VTAM's filter optimization explicitly processes negative control and mock samples to set thresholds
- The goal is a single-pipeline production analysis rather than a controlled cross-pipeline comparison under identical hardware and reference databases

## Inputs

- Raw paired-end amplicon FASTQ files for the bat dataset (Galan et al., 2018, 357 fecal pellet samples, Dryad doi:10.5061/dryad.f40v5)
- Raw paired-end amplicon FASTQ files for the fish dataset (Corse et al., 2017, 35 Zingel asper excrement samples, Dryad doi:10.5061/dryad.kv02g)
- Raw paired-end amplicon FASTQ files for the shark dataset (Esposito et al., 2022, 63 gut content samples, two sequencing runs pooled, OSF https://osf.io/3txv7/)
- COInr COI reference database and RDP Classifier 16S trainsetNo18 formatted for VTAM
- Mock community and negative control sample metadata for TP/FP classification
- Snakemake workflow files (01snkfl_prep.yml, 02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, 04snkfl_analysis_obibar.yml, 05snkfl_min_readcount.yml) and vtam_benchmark.def container definition

## Outputs

- Trimmed, demultiplexed fasta/fastq preparation outputs per dataset
- Per-pipeline ASV tables with taxonomic assignment (vtam taxassign, *_final_taxa.tsv) at min_readcount cut-offs 0, 10, 40, 60
- control_counts.tsv files classifying occurrences as true/false positives per pipeline and cut-off
- Sensitivity (TP/(TP+FN)) and precision (TP/(TP+FP)) values plotted with scripts/plt_sensitivity_precision.py
- runtime_memory_table.tsv with user/system/elapsed time, CPU %, and maxresident memory per Snakemake workflow

## How to apply

Clone the vtam_benchmark repository and build the Singularity container with 'sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def'. Download the bat and fish datasets from Dryad (doi:10.5061/dryad.f40v5, doi:10.5061/dryad.kv02g) and the shark dataset from OSF (https://osf.io/3txv7/), pooling the two shark runs with cat/gzip. Run the preparation workflow 'snakemake -s 01snkfl_prep.yml' (min_readcount=0) to produce trimmed, demultiplexed fasta/fastq via VTAM's merge and sortreads commands, then run the three analysis workflows: 02snkfl_analysis_vtam.yml (VTAM filter with per-dataset optimized parameters, e.g. lfn_read_count_cutoff 60 for bat/10 for fish/40 for shark, plus taxassign), 03snkfl_analysis_dalu.yml (DADA2 1.12.1 + LULU 0.1.0 + add_filters.pl), and 04snkfl_analysis_obibar.yml (OBITools3 3.0.1b19 + metabaR 1.0.0), each with COInr COI / RDP 16S trainsetNo18 references and --resources db_bat=1 db_fish=1 db_shark=1. Sweep minimum read count cut-offs 0/10/40/60 with 05snkfl_min_readcount.yml, classify true/false positives against control samples with count_false_occ, and compute sensitivity TP/(TP+FN) and precision TP/(TP+FP). Measure user/system time, elapsed time, CPU %, and maxresident memory (e.g. /usr/bin/time -v) on an 8-CPU 32 GB RAM desktop to compile runtime_memory_table.tsv.

## Related tools

- **VTAM** (Primary pipeline: merge, sortreads, filter (FilterLNF, FilterPCRerror, FilterMinReplicateNumber, FilterRenkonen, FilterChimera, FilterCodonStop, FilterIndel), optimize, and taxassign commands) — https://github.com/aitgon/vtam
- **DADA2** (Denoising engine of the DALU comparison pipeline (v1.12.1) with marker-specific truncation lengths)
- **LULU** (Post-denoising curation of DALU ASVs (v0.1.0))
- **OBITools3** (Denoising (obi uniq/grep/clean) engine of the OBIbaR comparison pipeline (v3.0.1b19))
- **metabaR** (Occurrence filtering and replicate pooling in the OBIbaR pipeline (v1.0.0))
- **cutadapt** (Primer/tag trimming in the preparation workflow)
- **vsearch** (VTAM dependency for sequence processing (e.g. chimera handling))
- **Snakemake** (Workflow orchestration of prep, analysis, and min_readcount cut-off sweep Snakemake files)
- **Singularity** (Containerizes the whole benchmark via vtam_benchmark.def for reproducibility)
- **Python** (VTAM implementation language and plotting scripts (scripts/plt_sensitivity_precision.py)) — https://github.com/aitgon/vtam

## Examples

```
sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def; snakemake -s 01snkfl_prep.yml --use-singularity --resources db_bat=1 db_fish=1 db_shark=1
```

## Evaluation signals

- All three pipelines run to completion without missing output files (rerun on snakemake latency-wait errors); control_counts.tsv and *_final_taxa.tsv exist for all pipelines at min_readcount 0, 10, 40, 60
- Runtime ordering reproduces the published pattern: VTAM fastest at 00:11:01, DALU at 00:57:53, OBIbaR at 01:01:43
- Maxresident memory falls near the reported values (2815 M VTAM, 1808 M DALU, 2689 M OBIbaR) on the same 8 CPU / 32 GB RAM hardware class
- VTAM reaches precision 1.00 on the five marine fish samples versus 0.66 (OBIbaR) and 0.19 (DALU), with sensitivity 0.95–1 for all pipelines
- Precision of DALU and OBIbaR increases monotonically with the minimum read count cut-off, matching the published trend

## Limitations

- VTAM's FilterCodonStop and FilterIndel are skipped for the shark dataset, so filter behavior is marker/dataset dependent
- Optimized VTAM parameters (lfn_read_count_cutoff, pcr_error_var_prop, lfn_sample_replicate_cutoff, lfn_variant_cutoff) are dataset-specific and must be re-derived via vtam optimize for new data
- Benchmark timing/memory values are only valid for the reference hardware class (8 CPU, 32 GB RAM desktop) and within plausible variance
- Control-sample-based filtering presupposes adequate mock communities; the article warns that metabarcoders studying metazoa or plants must construct representative mock samples

## Evidence
