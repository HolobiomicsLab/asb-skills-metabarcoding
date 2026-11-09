---
name: singularity-containerized-benchmarking
description: Use when a study needs to benchmark multiple bioinformatics pipelines
  on the same FASTQ datasets under identical software environments, e.g. comparing
  VTAM vs DADA2+LULU vs OBITools3+metabaR runtime and memory.
when_to_use_negative:
- You only need to run a single pipeline (e.g. VTAM alone) for data analysis rather
  than a comparative runtime/memory benchmark of the three pipelines
- Input is already a processed ASV/feature table — the benchmark operates on raw demultiplexed
  FASTQ reads (bat, fish, shark datasets)
- Hardware with fewer than the reference desktop resources (8 CPU, 32 GB RAM) is required
  for the exact runtime/memory comparison, since published values like maxresident
  2815 M for VTAM are hardware-dependent
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3174
tools:
- name: VTAM
  role: Pipeline under benchmark; its optimized filter parameters (e.g. lfn_read_count_cutoff
    60 for bat) are set in 02snkfl_analysis_vtam.yml
  repo: https://github.com/aitgon/vtam
- name: Snakemake
  role: Workflow manager executing the prep (01snkfl_prep.yml) and analysis (02-04snkfl_analysis_*.yml)
    workflows with the Singularity container and --resources db_bat/db_fish/db_shark
    limits
  repo: https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- name: Singularity
  role: Builds the identical software environment for all three pipelines from vtam_benchmark.def
- name: DADA2
  role: Denoising engine of the DALU pipeline (DADA2 + LULU + supplementary filters)
    benchmarked via 03snkfl_analysis_dalu.yml
- name: OBITools3
  role: Denoising engine of the OBIbaR pipeline (OBITools3 + metabaR + supplementary
    filters) benchmarked via 04snkfl_analysis_obibar.yml
- name: LULU
  role: Post-clustering curation step in the DALU pipeline used in the benchmark comparison
- name: metabaR
  role: Curation step in the OBIbaR pipeline used in the benchmark comparison
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:e099709738e9e939e96b192c9b600e121cf869dba3e609f7df95d08881965e76
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
  - Singularity
  - Snakemake
  - VTAM
  - metabaR
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# singularity-containerized-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rerun three metabarcoding pipelines (VTAM, DALU, OBIbaR) under an identical Singularity containerized Snakemake setup and measure their runtime and memory usage, reproducing the VTAM benchmark publication's elapsed-time and maxresident-memory comparison.

## When to use

Use when a study needs to benchmark multiple bioinformatics pipelines on the same FASTQ datasets under identical software environments, e.g. comparing VTAM vs DADA2+LULU vs OBITools3+metabaR runtime and memory.

## When NOT to use

- You only need to run a single pipeline (e.g. VTAM alone) for data analysis rather than a comparative runtime/memory benchmark of the three pipelines
- Input is already a processed ASV/feature table — the benchmark operates on raw demultiplexed FASTQ reads (bat, fish, shark datasets)
- Hardware with fewer than the reference desktop resources (8 CPU, 32 GB RAM) is required for the exact runtime/memory comparison, since published values like maxresident 2815 M for VTAM are hardware-dependent

## Inputs

- Bat dataset FASTQ files from Dryad (doi:10.5061/dryad.f40v5, file streams 64339-64342) unzipped into out/data_bat
- Fish dataset FASTQ files (MFZR1-3, ZFZR1-3 R1/R2) from Dryad doi:10.5061/dryad.kv02g (file streams 62829-62851) untarred into out/data_fish
- Shark dataset FASTQ files from OSF (https://osf.io/3txv7/, e.g. osf.io/ymswg, osf.io/q69de) pooled per sample with cat |gzip into out/data_shark
- COInr COI reference database and RDP Classifier 16S trainsetNo18 formatted for VTAM taxonomic assignment
- vtam_benchmark.def Singularity definition file from https://github.com/aitgon/vtam_benchmark

## Outputs

- out/vtam_benchmark.sif Singularity container image
- Trimmed, demultiplexed fasta/fastq preparation outputs from 01snkfl_prep.yml (min_readcount=0)
- Per-pipeline ASV tables and *_final_taxa.tsv files for VTAM, DALU, and OBIbaR from the three analysis Snakemake workflows
- out/summary_min_readcount_XX outputs (XX = 0, 10, 40, 60) including control_counts.tsv from 05snkfl_min_readcount.yml
- runtime_memory_table.tsv with elapsed time, user/system time, CPU %, and maxresident memory per Snakemake workflow

## How to apply

Clone the vtam_benchmark repository and build the pinned software environment with 'sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def', then run Snakemake against that container. Download the bat and fish FASTQ datasets from Dryad (doi:10.5061/dryad.f40v5, doi:10.5061/dryad.kv02g) into out/data_bat and out/data_fish, and the shark dataset from OSF (https://osf.io/3txv7/), pooling its two runs with cat/gzip into out/data_shark. Run the preparation workflow 01snkfl_prep.yml with min_readcount=0, then the three analysis workflows 02snkfl_analysis_vtam.yml (VTAM filter with optimized parameters, e.g. lfn_read_count_cutoff 60 for bat/10 for fish/40 for shark, plus taxassign against the COInr COI and RDP 16S trainsetNo18 databases), 03snkfl_analysis_dalu.yml (DADA2 + LULU), and 04snkfl_analysis_obibar.yml (OBITools3 + metabaR), using --resources db_bat=1 db_fish=1 db_shark=1 on an 8-CPU, 32 GB RAM desktop. Wrap each workflow with /usr/bin/time -v to capture user time, system time, elapsed time, CPU %, and maxresident memory, and compile the rows (01snkfl_prep.yml through 04snkfl_analysis_obibar.yml) into runtime_memory_table.tsv. The rationale is that a single Singularity image plus shared Snakemake resource limits removes software-environment and scheduling confounds so the pipelines' reported differences (VTAM fastest) are attributable to the pipelines themselves.

## Related tools

- **VTAM** (Pipeline under benchmark; its optimized filter parameters (e.g. lfn_read_count_cutoff 60 for bat) are set in 02snkfl_analysis_vtam.yml) — https://github.com/aitgon/vtam
- **Snakemake** (Workflow manager executing the prep (01snkfl_prep.yml) and analysis (02-04snkfl_analysis_*.yml) workflows with the Singularity container and --resources db_bat/db_fish/db_shark limits) — https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- **Singularity** (Builds the identical software environment for all three pipelines from vtam_benchmark.def)
- **DADA2** (Denoising engine of the DALU pipeline (DADA2 + LULU + supplementary filters) benchmarked via 03snkfl_analysis_dalu.yml)
- **OBITools3** (Denoising engine of the OBIbaR pipeline (OBITools3 + metabaR + supplementary filters) benchmarked via 04snkfl_analysis_obibar.yml)
- **LULU** (Post-clustering curation step in the DALU pipeline used in the benchmark comparison)
- **metabaR** (Curation step in the OBIbaR pipeline used in the benchmark comparison)

## Examples

```
sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def && snakemake -s 02snkfl_analysis_vtam.yml --resources db_bat=1 db_fish=1 db_shark=1
```

## Evaluation signals

- Elapsed-time ordering reproduces the published pattern: VTAM fastest at 00:11:01, DALU 00:57:53, OBIbaR 01:01:43, with prep workflow completing first
- Maxresident memory falls within plausible variance of published values on the same hardware class: 2815 M (VTAM), 1808 M (DALU), 2689 M (OBIbaR)
- Each Snakemake workflow runs to completion without missing-output errors; latency-wait errors are resolved by rerunning rather than manual fixes
- All four min_readcount runs (0, 10, 40, 60) produce out/summary_min_readcount_XX with control_counts.tsv and *_final_taxa.tsv files for all three pipelines
- runtime_memory_table.tsv contains rows for 01snkfl_prep.yml, 02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, and 04snkfl_analysis_obibar.yml with user time, system time, elapsed time, CPU %, and maxresident memory

## Limitations

- Published runtime/memory values are specific to the reference hardware (8 CPU, 32 GB RAM desktop); different hardware yields different absolute times and memory figures
- Software versions of DADA2, OBITools, VTAM, and metabaR are not stated, so exact version pinning must come from the vtam_benchmark.def container definition
- Snakemake latency-wait errors on missing output files can occur and require reruns; intermediate directories such as out/vtam_fish/merged_mfzr and out/vtam_shark/merged must be removed after prep as instructed by the README

## Evidence
