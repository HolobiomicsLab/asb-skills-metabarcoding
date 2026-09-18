---
name: snakemake-workflow-execution
description: Use when raw paired-end amplicon FASTQ files must be processed end-to-end
  into ASV tables and taxonomic assignments, and you need a single-command, cluster-aware
  Snakemake workflow with dry-run validation before committing compute.
when_to_use_negative:
- Input is already a processed feature/ASV table rather than raw FASTQ files — dadasnake
  starts from raw fastq-files.
- You need interactive, step-by-step control over individual DADA2 parameter tuning
  rather than the configurable one-command workflow.
- The computing environment lacks conda, since dadasnake requires conda for installation
  and its single-environment execution model.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: dadasnake
  role: Single-command wrapper that validates, configures, and launches the Snakemake
    workflow; provides dry-run (-d), local (-l), and cluster-submit (-c) modes and
    emits the workflow.done completion marker.
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: Workflow engine dispatched by dadasnake to parallelize per-sample DADA2 preprocessing
    and denoising jobs across cluster cores (Snakemake >= 5.9.1, tested version 6.9.1).
  repo: https://github.com/a-h-b/dadasnake
- name: conda
  role: Required for installation and for provisioning the single conda environment
    (optionally including Snakemake itself via SNAKEMAKE_VIA_CONDA) in which all workflow
    steps run.
- name: tmux
  role: Used by the dadasnake_tmux wrapper to keep the Snakemake top instance running
    detached on the cluster frontend when not submitting it as a job.
  repo: https://github.com/tmux/tmux/wiki
merged_aliases:
- snakemake-cluster-workflow-execution
- snakemake-workflow-configuration-and-execution
merged_alias_records:
- alias: snakemake-cluster-workflow-execution
  slug: snakemake-cluster-workflow-execution
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
- alias: snakemake-workflow-configuration-and-execution
  slug: snakemake-workflow-configuration-and-execution
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:485e957507d9cca9be46dc4b4affab51d47bdc5d7659f6aec94986017c999105
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  - tmux
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# snakemake-workflow-execution

## Summary

Execute the dadasnake Snakemake pipeline that drives DADA2-based amplicon preprocessing, ASV delineation, and taxonomic classification from a single command, validating inputs first via dry-run and dispatching jobs to a cluster or local cores with runtime-configurable thread counts.

## When to use

Use when raw paired-end amplicon FASTQ files must be processed end-to-end into ASV tables and taxonomic assignments, and you need a single-command, cluster-aware Snakemake workflow with dry-run validation before committing compute.

## When NOT to use

- Input is already a processed feature/ASV table rather than raw FASTQ files — dadasnake starts from raw fastq-files.
- You need interactive, step-by-step control over individual DADA2 parameter tuning rather than the configurable one-command workflow.
- The computing environment lacks conda, since dadasnake requires conda for installation and its single-environment execution model.

## Inputs

- raw paired-end amplicon FASTQ files in a single raw_directory (e.g. ERR777696 fastqs or PRJNA517390 reads)
- YAML configuration file specifying raw_directory, sample_table, output directory, paired=true, trunc_length, and database paths (e.g. SILVA v.138)
- tab-separated sample table with 'library', 'r1_file', 'r2_file' headers and run IDs

## Outputs

- ASV count tables (tab-separated, biom format, R/phyloseq objects)
- taxonomically classified ASV sequences (fasta and tables via mothur/BLAST+BASTA)
- intermediary step statistics in figures and tables
- workflow.done marker file in the output directory upon completion

## How to apply

Clone the dadasnake repository and copy one of the wrappers (dadasnake_allSubmit for cluster submission or dadasnake_tmux to keep the Snakemake top instance running via tmux on the frontend), then adjust VARIABLE_CONFIG: SCHEDULER (slurm or uge), MAX_THREADS (default 50), NORMAL_MEM_EACH, and optionally SNAKEMAKE_VIA_CONDA to install Snakemake 6.9.1 in a conda environment. Write a human-readable YAML config (output directory, raw_directory, sample_table path, paired=true, trunc_length) and a tab-separated sample table with 'library', 'r1_file', and 'r2_file' columns plus run IDs so run-specific DADA2 error models can be estimated. Always validate first with a dry-run ('dadasnake -d -r config.yaml') to confirm input paths, sample table formatting, and write permissions before launching the real run ('dadasnake -l -n RUNNAME -r config.yaml' locally, or 'dadasnake -c -n RUNNAME -r config.yaml -t 50' in cluster-submit mode so Snakemake dispatches jobs to at most 50 cores). Monitor progress via the RUNNAME stderr file and per-rule log files; completion is signaled by a 'workflow.done' marker file appearing in the output directory. This design lets Snakemake efficiently parallelize per-sample DADA2 steps across a computing cluster while running everything from a single conda environment.

## Related tools

- **dadasnake** (Single-command wrapper that validates, configures, and launches the Snakemake workflow; provides dry-run (-d), local (-l), and cluster-submit (-c) modes and emits the workflow.done completion marker.) — https://github.com/a-h-b/dadasnake
- **Snakemake** (Workflow engine dispatched by dadasnake to parallelize per-sample DADA2 preprocessing and denoising jobs across cluster cores (Snakemake >= 5.9.1, tested version 6.9.1).) — https://github.com/a-h-b/dadasnake
- **conda** (Required for installation and for provisioning the single conda environment (optionally including Snakemake itself via SNAKEMAKE_VIA_CONDA) in which all workflow steps run.)
- **tmux** (Used by the dadasnake_tmux wrapper to keep the Snakemake top instance running detached on the cluster frontend when not submitting it as a job.) — https://github.com/tmux/tmux/wiki

## Examples

```
dadasnake -d -r config.yaml; dadasnake -c -n RUNNAME -r config.yaml -t 50
```

## Evaluation signals

- Dry-run ('dadasnake -d -r config.yaml') completes without errors, confirming input paths, sample table formatting, and write permissions.
- A 'workflow.done' file appears in the output directory, signaling workflow completion.
- Benchmark: 24-sample 16S V4 dataset (2.9M reads) finishes in ~150 min on 1 core and ~100 min on 4 cores; EMP-scale (2.1B reads, >27,000 samples) finishes in ~87 real hours on <=50 cores.
- Final hand-off files are present in the expected formats: biom, phyloseq objects, fasta, and tab-separated tables.
- Per-rule log files and the RUNNAME stderr file show no failed jobs and intermediate statistics figures/tables are generated.

## Limitations

- Snakemake version must be >= 5.9.1; dadasnake has not been tested with previous versions.
- Runtime thread limits depend on VARIABLE_CONFIG settings (MAX_THREADS default 50), and users can override thread count only if LOCK_SETTINGS is not set to true.
- On slow file systems, extra Snakemake arguments such as --latency-wait=320 may be needed via SNAKEMAKE_EXTRA_ARGUMENTS.
- Cluster-specific schedulers are limited to slurm or uge out of the box; other systems require contacting the maintainers.

## Evidence

- [abstract] By use of Snakemake, dadasnake makes efficient use of high-performance computing infrastructures.: "By use of Snakemake, dadasnake makes efficient use of high-performance computing infrastructures."
- [intro] It is designed to run on a computing cluster using a single conda environment in multiple jobs triggered by Snakemake: "It is designed to run on a computing cluster using a single conda environment in multiple jobs triggered by Snakemake"
- [methods] git clone https://github.com/a-h-b/dadasnake.git: "have [tmux](https://github.com/tmux/tmux/wiki) installed"
- [readme] If you don't submit jobs to the cluster, but want to run the whole workflow interactively, e.g. on a laptop, it doesn't matter which wrapper you use.: "If you don't submit jobs to the cluster, but want to run the whole workflow interactively, e.g. on a laptop, it doesn't matter which wrapper you use."
- [readme] Dadasnake will run with Snakemake version >= 5.9.1 and hasn't been tested with any previous versions.: "Dadasnake will run with Snakemake version >= 5.9.1 and hasn't been tested with any previous versions."
