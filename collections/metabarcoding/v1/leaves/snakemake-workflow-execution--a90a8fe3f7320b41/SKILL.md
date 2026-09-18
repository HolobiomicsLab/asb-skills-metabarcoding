---
name: snakemake-workflow-execution
description: Use when reproducing the vtam_benchmark Snakemake workflows (01snkfl_prep.yml
  through 05snkfl_min_readcount.yml) on the bat, fish, and shark FASTQ datasets, or
  when coordinating resource-limited, containerized metabarcoding pipeline runs.
when_to_use_negative:
- When the target pipeline is a single self-contained script rather than a modular
  multi-rule workflow with per-rule inputs/outputs — a plain shell script is simpler
- When VTAM is run standalone via its own WopMars workflow manager (github.com/aitgon/wopmars)
  instead of the benchmark's Snakemake setup
- When datasets or reference databases differ from the three benchmark datasets (bat,
  fish, shark) and COI/16S references, since optimized parameters and cut-offs are
  dataset-specific
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow engine executing the prep, VTAM, DALU, OBIbaR, and min-readcount
    Snakefiles with --resources and --cores constraints
  repo: https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- name: Singularity
  role: Container runtime; the vtam_benchmark.sif image built from vtam_benchmark.def
    provides all pipeline dependencies reproducibly
- name: VTAM
  role: One of the three benchmarked pipelines, run via 02snkfl_analysis_vtam.yml
    (filter with optimized parameters plus taxassign)
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Language in which VTAM and the workflow tooling are implemented
  repo: https://github.com/aitgon/vtam
merged_aliases:
- snakemake-workflow-orchestration
merged_alias_records:
- alias: snakemake-workflow-orchestration
  slug: snakemake-workflow-orchestration
  cosine_score: 0.883
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:460f7e1342627d50fc85b8b85ab68200be8eb8ae58462144c6d6031bb3c36b4f
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
  - R
  - Singularity
  - Snakemake
  - VTAM
  - cutadapt
  - metabaR
  - vsearch
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# snakemake-workflow-execution

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Orchestrates the multi-step VTAM vs DALU vs OBIbaR metabarcoding benchmark (prep, three analysis pipelines, and min-read-count sweeps) as Snakemake workflows run inside a Singularity container, enabling reproducible execution and runtime/memory measurement of each pipeline.

## When to use

Use when reproducing the vtam_benchmark Snakemake workflows (01snkfl_prep.yml through 05snkfl_min_readcount.yml) on the bat, fish, and shark FASTQ datasets, or when coordinating resource-limited, containerized metabarcoding pipeline runs.

## When NOT to use

- When the target pipeline is a single self-contained script rather than a modular multi-rule workflow with per-rule inputs/outputs — a plain shell script is simpler
- When VTAM is run standalone via its own WopMars workflow manager (github.com/aitgon/wopmars) instead of the benchmark's Snakemake setup
- When datasets or reference databases differ from the three benchmark datasets (bat, fish, shark) and COI/16S references, since optimized parameters and cut-offs are dataset-specific

## Inputs

- Snakemake workflow files: 01snkfl_prep.yml, 02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, 04snkfl_analysis_obibar.yml, 05snkfl_min_readcount.yml
- Singularity definition file vtam_benchmark.def
- Raw paired-end FASTQ datasets: bat (Dryad doi:10.5061/dryad.f40v5), fish (Dryad doi:10.5061/dryad.kv02g), shark (OSF https://osf.io/3txv7/, pooled from two runs with cat/gzip)
- Reference databases: COInr COI and RDPClassifier_16S_trainsetNo18 formatted for VTAM

## Outputs

- Trimmed, demultiplexed fasta/fastq files under out/data_bat, out/data_fish, out/data_shark directories
- out/summary_min_readcount_XX directories (XX = 0, 10, 40, 60) containing control_counts.tsv and *_final_taxa.tsv for all three pipelines
- runtime_memory_table.tsv with user time, system time, elapsed time, CPU %, and maxresident memory per workflow
- out/vtam_benchmark.sif Singularity container image

## How to apply

Clone the vtam_benchmark repository, build the container with 'sudo singularity build out/vtam_benchmark.sif vtam_benchmark.def', and install Snakemake in a dedicated conda environment (mamba create -c conda-forge -c bioconda -n snakemake snakemake). Run 01snkfl_prep.yml with min_readcount=0 to produce trimmed, demultiplexed fasta/fastq files, then execute the three analysis workflows (02snkfl_analysis_vtam.yml, 03snkfl_analysis_dalu.yml, 04snkfl_analysis_obibar.yml) with --resources db_bat=1 db_fish=1 db_shark=1 so COInr COI and RDP 16S trainsetNo18 database-heavy jobs are serialized. Sweep the minimum read count cut-off over 0, 10, 40, and 60 by running 05snkfl_min_readcount.yml four times with min_readcount set to each value, mirroring VTAM's optimized suggested cut-offs (10 for fish, 40 for shark, 60 for bat). Wrap prep and analysis runs with /usr/bin/time -v on an 8-CPU, 32 GB RAM desktop to capture user/system time, elapsed time, CPU %, and maxresident memory for the runtime_memory_table.tsv comparison.

## Related tools

- **Snakemake** (Workflow engine executing the prep, VTAM, DALU, OBIbaR, and min-readcount Snakefiles with --resources and --cores constraints) — https://snakemake.readthedocs.io/en/stable/getting_started/installation.html
- **Singularity** (Container runtime; the vtam_benchmark.sif image built from vtam_benchmark.def provides all pipeline dependencies reproducibly)
- **VTAM** (One of the three benchmarked pipelines, run via 02snkfl_analysis_vtam.yml (filter with optimized parameters plus taxassign)) — https://github.com/aitgon/vtam
- **Python** (Language in which VTAM and the workflow tooling are implemented) — https://github.com/aitgon/vtam

## Examples

```
snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- Each Snakemake workflow runs to completion without 'missing output files' errors; rerun when snakemake latency-wait errors occur
- All four out/summary_min_readcount_XX directories (XX = 0, 10, 40, 60) exist and contain control_counts.tsv plus *_final_taxa.tsv for VTAM, DALU, and OBIbaR
- Elapsed-time ordering reproduces the published pattern: VTAM fastest (00:11:01), then DALU (00:57:53), then OBIbaR (01:01:43), with maxresident memory of 2815 M (VTAM), 1808 M (DALU), and 2689 M (OBIbaR) within plausible variance on the same hardware class (8 CPU, 32 GB RAM)
- The Singularity image out/vtam_benchmark.sif builds successfully from vtam_benchmark.def before any workflow is launched
- Intermediate directories listed in the README (e.g. out/vtam_fish/merged_mfzr, out/vtam_shark/merged) are removed after the prep workflow as instructed

## Limitations

- The two shark sequencing runs must first be pooled with cat and gzip-ped before the workflows can run; raw OSF archives are not directly consumable
- Resource flags (--resources db_bat=1 db_fish=1 db_shark=1) are required to prevent concurrent database-heavy taxassign jobs from exhausting memory
- Runtime and memory figures are hardware-dependent and only comparable within the reference desktop class used in the publication
- Snakemake latency-wait errors require manual reruns of the affected workflow

## Evidence

- [readme] snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa: [evidence span withheld — anchor status: external_doc]
