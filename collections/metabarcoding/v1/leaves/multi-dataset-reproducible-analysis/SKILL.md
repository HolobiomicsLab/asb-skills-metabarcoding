---
name: multi-dataset-reproducible-analysis
description: Use when the goal is to rerun several metabarcoding pipelines (VTAM,
  DALU, OBIbaR) on the same public datasets under identical conditions and reproduce
  published runtime/memory or precision comparisons.
when_to_use_negative:
- You only need to run a single pipeline (e.g. VTAM alone) on your own data — use
  the VTAM package directly instead of the benchmark harness
- Your goal is a methodological evaluation of filtering performance (sensitivity/precision)
  rather than reproducing the runtime and memory benchmark on the shared datasets
- Your datasets or hardware differ substantially from the three published datasets
  (bat, fish, shark) and the 8-CPU/32-GB reference desktop, making published timing
  values non-comparable
tools:
- name: VTAM
  role: Metabarcoding analysis pipeline under benchmark; also provides merge, sortreads,
    filter, optimize and taxassign commands used within the workflows
  repo: https://github.com/aitgon/vtam
- name: Snakemake
  role: Workflow manager orchestrating the prep and analysis Snakefiles with shared
    resource limits (db_bat/db_fish/db_shark)
  repo: https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- name: Singularity
  role: Container runtime used to build vtam_benchmark.sif so all three pipelines
    run under the same software environment
- name: DADA2
  role: Denoising step of the DALU pipeline, combined with LULU and supplementary
    filters
- name: OBITools3
  role: Denoising step of the OBIbaR pipeline, enhanced by metabaR and supplementary
    filters
- name: Cutadapt
  role: Dependency of VTAM used for primer trimming in the shared preparation workflow
  repo: https://github.com/aitgon/vtam
- name: Vsearch
  role: Dependency of VTAM installed via bioconda for sequence processing
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:968c4ad505aef01e50c38292b451c8852eda9e026b3cddafaa99fe10e4b96c86
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
  - Cutadapt
  - DADA2
  - LULU
  - OBITools3
  - Python
  - Singularity
  - Snakemake
  - VTAM
  - Vsearch
  - metabaR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# multi-dataset-reproducible-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce a published multi-pipeline metabarcoding benchmark by running the VTAM, DALU, and OBIbaR workflows via the Snakemake/Singularity-based vtam_benchmark repository on shared bat, fish, and shark datasets, then compare runtime and memory across pipelines.

## When to use

Use when the goal is to rerun several metabarcoding pipelines (VTAM, DALU, OBIbaR) on the same public datasets under identical conditions and reproduce published runtime/memory or precision comparisons.

## When NOT to use

- You only need to run a single pipeline (e.g. VTAM alone) on your own data — use the VTAM package directly instead of the benchmark harness
- Your goal is a methodological evaluation of filtering performance (sensitivity/precision) rather than reproducing the runtime and memory benchmark on the shared datasets
- Your datasets or hardware differ substantially from the three published datasets (bat, fish, shark) and the 8-CPU/32-GB reference desktop, making published timing values non-comparable

## Inputs

- Raw FASTQ data for the bat dataset (Galan et al., 2018) downloaded from Dryad
- Raw FASTQ data for the fish dataset (Corse et al., 2017, Zingel asper excrement samples) downloaded from Dryad
- Raw FASTQ data for the shark dataset (Esposito et al., 2022, 63 gut content samples) downloaded from OSF (https://osf.io/3txv7/)
- COInr COI reference database
- RDP Classifier 16S trainsetNo18 reference database formatted for VTAM
- vtam_benchmark Snakemake workflow files and Singularity definition file

## Outputs

- Trimmed, demultiplexed fasta/fastq files from the preparation workflow (out/data_bat, out/data_fish, out/data_shark)
- Per-pipeline ASV tables with taxonomic assignments (*_final_taxa.tsv for VTAM, DALU, and OBIbaR)
- out/summary_min_readcount_XX outputs (XX = 00, 10, 40, 60) including control_counts.tsv
- runtime_memory_table.tsv with user time, system time, elapsed time, CPU %, and maxresident memory per Snakemake workflow

## How to apply

Clone the vtam_benchmark repository and build the provided Singularity container with 'sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def'. Download the bat and fish datasets from Dryad (doi:10.5061/dryad.f40v5 and doi:10.5061/dryad.kv02g) and the shark dataset from OSF, unzip/untar them into out/data_bat, out/data_fish, and out/data_shark, and pool the two shark runs with cat/gzip. Run the preparation workflow 01snkfl_prep.yml with min_readcount=0, then execute the three analysis workflows (02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, 04snkfl_analysis_obibar.yml) using the COInr COI and RDP 16S trainsetNo18 reference databases with --resources db_bat=1 db_fish=1 db_shark=1, applying VTAM's optimized per-dataset lfn_read_count_cutoff values (60 for bat, 10 for fish, 40 for shark). Measure user/system/elapsed time, CPU %, and maxresident memory (e.g. /usr/bin/time -v) for each workflow on the reference hardware class (8 CPU, 32 GB RAM desktop), and compile results into runtime_memory_table.tsv.

## Related tools

- **VTAM** (Metabarcoding analysis pipeline under benchmark; also provides merge, sortreads, filter, optimize and taxassign commands used within the workflows) — https://github.com/aitgon/vtam
- **Snakemake** (Workflow manager orchestrating the prep and analysis Snakefiles with shared resource limits (db_bat/db_fish/db_shark)) — https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- **Singularity** (Container runtime used to build vtam_benchmark.sif so all three pipelines run under the same software environment)
- **DADA2** (Denoising step of the DALU pipeline, combined with LULU and supplementary filters)
- **OBITools3** (Denoising step of the OBIbaR pipeline, enhanced by metabaR and supplementary filters)
- **Cutadapt** (Dependency of VTAM used for primer trimming in the shared preparation workflow) — https://github.com/aitgon/vtam
- **Vsearch** (Dependency of VTAM installed via bioconda for sequence processing) — https://github.com/aitgon/vtam

## Examples

```
sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def
```

## Evaluation signals

- Elapsed-time ordering reproduces the reported pattern: VTAM fastest (00:11:01), followed by DALU (00:57:53) and OBIbaR (01:01:43)
- Maxresident memory values (2815 M for VTAM, 1808 M for DALU, 2689 M for OBIbaR) are recovered within plausible variance on the same hardware class
- All three pipelines run to completion without missing output files (rerun on Snakemake latency-wait errors)
- Running 05snkfl_min_readcount.yml with min_readcount 0, 10, 40, and 60 yields all four out/summary_min_readcount_XX outputs including control_counts.tsv and *_final_taxa.tsv for every pipeline
- VTAM filter parameters match the optimized values (e.g. lfn_read_count_cutoff 60 for bat, 10 for fish, 40 for shark; lfn_sample_replicate_cutoff 0.002 and pcr_error_var_prop 0.2 for bat)

## Limitations

- Runtime and memory figures are hardware-class dependent (reference: 8 CPU, 32 GB RAM desktop); comparisons on other machines will not match the published values exactly
- Taxonomic assignments were done with VTAM's taxassign command for all three pipelines to ensure homogeneity, so per-pipeline native assigners are not compared
- Software versions of DADA2, OBITools, VTAM, and metabaR are not stated, and Snakemake latency-wait errors may require reruns of affected jobs

## Evidence
