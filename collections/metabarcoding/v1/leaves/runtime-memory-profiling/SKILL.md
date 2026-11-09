---
name: runtime-memory-profiling
description: Use when reproducing the VTAM vs DALU vs OBIbaR benchmark and you need
  to measure and compare elapsed time and maxresident memory of each Snakemake workflow
  (01snkfl_prep.yml through 04snkfl_analysis_obibar.yml) on the same hardware.
when_to_use_negative:
- When the goal is to compare pipeline accuracy or precision (sensitivity, ASV richness,
  false positives) rather than compute cost — that requires the min_readcount sensitivity
  workflows, not the timing wrapper.
- When the pipelines cannot be run under the same Singularity container and same hardware
  class, since cross-hardware timing and maxresident memory values are not comparable.
tools:
- name: Snakemake
  role: Executes the prep and analysis workflows that are being profiled
  repo: https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- name: Singularity
  role: Containerizes the benchmark so all three pipelines run under the identical
    environment during timing
- name: VTAM
  role: Fastest benchmarked pipeline (00:11:01, 2815 M maxresident); its analysis
    workflow 02snkfl_analysis_vtam.yml is one of the profiled jobs
  repo: https://github.com/aitgon/vtam
- name: vtam_benchmark
  role: Repository providing the Snakemake workflows and container definition needed
    to reproduce the runtime/memory comparison
  repo: https://github.com/aitgon/vtam_benchmark
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:b7a41ac04439bdd04cd3568ba59fc9b29865354478f039ba2824328b3839821f
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aitgon/vtam_benchmark
  tools:
  - DADA2
  - LULU
  - OBITools3
  - Python
  - Singularity
  - Snakemake
  - VTAM
  - metabaR
  - vtam_benchmark
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# runtime-memory-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Profiles the compute cost of the three metabarcoding pipelines (VTAM, DALU, OBIbaR) by measuring user time, system time, elapsed time, CPU %, and maxresident memory for each Snakemake workflow in the vtam_benchmark repository on a fixed 8 CPU / 32 GB RAM desktop, producing a runtime_memory_table.tsv that reproduces the published comparison.

## When to use

Use when reproducing the VTAM vs DALU vs OBIbaR benchmark and you need to measure and compare elapsed time and maxresident memory of each Snakemake workflow (01snkfl_prep.yml through 04snkfl_analysis_obibar.yml) on the same hardware.

## When NOT to use

- When the goal is to compare pipeline accuracy or precision (sensitivity, ASV richness, false positives) rather than compute cost — that requires the min_readcount sensitivity workflows, not the timing wrapper.
- When the pipelines cannot be run under the same Singularity container and same hardware class, since cross-hardware timing and maxresident memory values are not comparable.

## Inputs

- Snakemake workflow files: 01snkfl_prep.yml, 02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, 04snkfl_analysis_obibar.yml
- Singularity container vtam_benchmark.sif built from vtam_benchmark.def
- Bat and fish datasets from Dryad (doi:10.5061/dryad.f40v5 and doi:10.5061/dryad.kv02g) and pooled shark dataset from OSF (https://osf.io/3txv7/) placed in out/data_bat, out/data_fish, out/data_shark
- COInr COI and RDP Classifier 16S trainsetNo18 reference databases
- Reference hardware: 8 CPU, 32 GB RAM desktop

## Outputs

- runtime_memory_table.tsv with rows for 01snkfl_prep.yml, 02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, 04snkfl_analysis_obibar.yml
- Per-workflow user time, system time, elapsed time, CPU %, and maxresident memory measurements (from /usr/bin/time -v)

## How to apply

After cloning https://github.com/aitgon/vtam_benchmark and building the container with 'sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def', run the preparation workflow 01snkfl_prep.yml (min_readcount=0) and the three analysis workflows 02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, and 04snkfl_analysis_obibar.yml via Snakemake with the COInr COI and RDP 16S trainsetNo18 databases and --resources db_bat=1 db_fish=1 db_shark=1. Wrap each Snakemake invocation in /usr/bin/time -v so that user time, system time, elapsed time, CPU %, and maxresident memory are captured for each workflow. Run the measurement on the reference hardware class (8 CPU, 32 GB RAM desktop) and compile rows for each of the four workflows into runtime_memory_table.tsv. The rationale is that only wall-clock and memory measured under identical hardware, container, and data conditions (bat, fish, and pooled shark datasets) are comparable across the three pipelines.

## Related tools

- **Snakemake** (Executes the prep and analysis workflows that are being profiled) — https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- **Singularity** (Containerizes the benchmark so all three pipelines run under the identical environment during timing)
- **VTAM** (Fastest benchmarked pipeline (00:11:01, 2815 M maxresident); its analysis workflow 02snkfl_analysis_vtam.yml is one of the profiled jobs) — https://github.com/aitgon/vtam
- **vtam_benchmark** (Repository providing the Snakemake workflows and container definition needed to reproduce the runtime/memory comparison) — https://github.com/aitgon/vtam_benchmark

## Examples

```
sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def && snakemake -s 01snkfl_prep.yml --cores 8 --use-singularity --resources db_bat=1 db_fish=1 db_shark=1
```

## Evaluation signals

- Elapsed-time ordering reproduces the reported pattern: VTAM fastest at 00:11:01, followed by DALU at 00:57:53 and OBIbaR at 01:01:43.
- Maxresident memory falls near the reported values: 2815 M for VTAM, 1808 M for DALU, 2689 M for OBIbaR, within plausible variance for the same 8 CPU / 32 GB RAM hardware class.
- runtime_memory_table.tsv contains a row for each of the four workflows (01snkfl_prep.yml through 04snkfl_analysis_obibar.yml) with user time, system time, elapsed time, CPU %, and maxresident memory filled in.
- Each pipeline runs to completion without missing output files; snakemake latency-wait errors trigger a rerun rather than a truncated measurement.

## Limitations

- Reported timings and memory figures are specific to the reference hardware class (8 CPU, 32 GB RAM desktop); other machines will yield different values.
- Timings assume the optimized VTAM parameters per dataset (e.g. lfn_read_count_cutoff 60 for bat, 10 for fish, 40 for shark) and the same reference databases; changing parameters or databases changes runtime.
- The README does not state software versions of DADA2, OBITools, VTAM, and metabaR, so exact reproducibility of runtimes may drift with newer tool versions.

## Evidence
