---
name: snakemake-workflow-execution
description: Use when running or reproducing the Tourmaline QIIME 2 amplicon workflow
  end-to-end, timing per-step wall-clock runtimes with different --cores values, or
  re-running individual rules on new parameter settings.
when_to_use_negative:
- Data was produced by an external program and you cannot supply the three Tourmaline
  config files or QIIME 2 artifacts — Tourmaline 1 cannot start mid-workflow, so direct
  QIIME 2 commands are needed instead
- The task is single-command QC on raw reads only — running the full Snakemake DAG
  is unnecessary overhead versus invoking the qaqc step alone
- Non-Illumina amplicon data, since Tourmaline 2 is scoped to Illumina sequence data
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: Snakemake
  role: Workflow management system that runs individual rules and manages step ordering,
    inputs, and outputs
  repo: github.com/aomlomics/tourmaline
- name: Tourmaline
  role: The amplicon workflow being executed; tourmaline.sh wraps the three steps
    and sets core count
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Engine invoked by Snakemake rules for denoising, taxonomy, and diversity
- name: Docker
  role: Container runtime for the workflow on systems without native conda setup
provenance:
  source_task_ids:
  - task_001
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:b68e6e5ba66b361eeff5478b53b5521d1767dc5e18cff0a1844620d422fe4084
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Docker
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# snakemake-workflow-execution

## Summary

Executing the Tourmaline amplicon pipeline through Snakemake rules or the tourmaline.sh wrapper, which orchestrates QIIME 2 commands (qaqc, repseqs, taxonomy steps) from three step-specific config files with multi-core support.

## When to use

Use when running or reproducing the Tourmaline QIIME 2 amplicon workflow end-to-end, timing per-step wall-clock runtimes with different --cores values, or re-running individual rules on new parameter settings.

## When NOT to use

- Data was produced by an external program and you cannot supply the three Tourmaline config files or QIIME 2 artifacts — Tourmaline 1 cannot start mid-workflow, so direct QIIME 2 commands are needed instead
- The task is single-command QC on raw reads only — running the full Snakemake DAG is unnecessary overhead versus invoking the qaqc step alone
- Non-Illumina amplicon data, since Tourmaline 2 is scoped to Illumina sequence data

## Inputs

- Three step config files (qaqc, repseqs, taxonomy) defining parameters and input paths
- Raw paired-end FASTQ archive or sample manifest for PRJNA679730 Lake Erie 16S data
- Sample metadata file
- Reference database (e.g., silva-138_1 or PR2)
- Step-specific Snakefile for individually invoked rules

## Outputs

- DADA2/Deblur feature table and representative-sequence QIIME 2 artifacts
- Taxonomy assignments for representative sequences
- Taxonomic, alpha, and beta diversity outputs
- HTML report (e.g., report_dada2-pe_unfiltered.html) linking web-viewable QIIME 2 visualizations
- runtimes.csv with per-step and total timings

## How to apply

Activate the snakemake-tour2 conda environment and either call ./tourmaline.sh with --step (qaqc, repseqs, taxonomy), --configfile, and --cores N, or run individual Snakemake rules — each step has its own Snakefile, so the correct snakefile must be specified. The qaqc step runs DADA2 or Deblur denoising (with optional Cutadapt primer trimming); repseqs applies representative-sequence filters (taxonomy, length, abundance, prevalence, ID) via parameters like repseq_min_length and repseq_min_abundance; taxonomy assigns classification using a reference database (e.g., SILVA 138.1 or PR2). For benchmarking, run each step separately with --cores 1, record wall-clock time per step, then re-run end-to-end with --cores 8; the full Lake Erie PRJNA679730 run (~96 samples, ~11.5 million paired reads) took ~5 h on 1 core vs. ~2 h on 8 cores. Compile per-step and total timings into runtimes.csv.

## Related tools

- **Snakemake** (Workflow management system that runs individual rules and manages step ordering, inputs, and outputs) — github.com/aomlomics/tourmaline
- **Tourmaline** (The amplicon workflow being executed; tourmaline.sh wraps the three steps and sets core count) — github.com/aomlomics/tourmaline
- **QIIME 2** (Engine invoked by Snakemake rules for denoising, taxonomy, and diversity)
- **Docker** (Container runtime for the workflow on systems without native conda setup)

## Examples

```
conda activate snakemake-tour2; ./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N
```

## Evaluation signals

- Each Snakemake rule completes without error and the run directory contains feature table, rep-seqs, taxonomy, diversity outputs, and the linked HTML report
- Full-dataset runtime with --cores 8 (~2 h) is proportionally faster than --cores 1 (~5 h) on ~11.5M paired reads
- report_dada2-pe_unfiltered.html exists and summarizes parameters, metadata, inputs, and outputs of the unfiltered paired-end run
- Individual rule invocations use the Snakefile belonging to the matching step (qaqc, repseqs, or taxonomy)
- Per-step timings in runtimes.csv sum to approximately the reported total wall-clock time

## Limitations

- tourmaline.sh must be run from inside the tourmaline directory, though output destinations can be set anywhere
- Tourmaline 1 (V1 branch) requires running through the whole workflow order; mid-pipeline entry with external data is a Tourmaline 2 feature
- Runtime benchmarks are dataset-dependent: ~96 samples, ~11.5 million paired reads of Lake Erie 16S data inside the Docker container

## Evidence

- [readme] Tourmaline 2 manages commands, inputs, and outputs using the Snakemake workflow management system.: [evidence span withheld — anchor status: not_found]
- [readme] tourmaline.sh usage: "conda activate snakemake-tour2
./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [readme] individual rules need the correct Snakefile: "You can still run individual snakemake rules as before. Each of the three steps (explained more below) has its own Snakefile"
- [abstract] built using Snakemake workflow management system: "Tourmaline, a Python-based workflow that implements QIIME 2 and is built using the Snakemake workflow management system"
- [methods] clone the repository: "git clone https://github.com/aomlomics/tourmaline.git"
