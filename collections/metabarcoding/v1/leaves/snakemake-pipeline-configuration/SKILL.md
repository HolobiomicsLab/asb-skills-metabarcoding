---
name: snakemake-pipeline-configuration
description: Use when raw paired-end FASTQ amplicon reads (e.g. ITS2 or 16S) must
  be processed with dadasnake and the pipeline needs to be installed, configured for
  a cluster or laptop, and dry-run validated before execution.
when_to_use_negative:
- Input is already an ASV/OTU or feature table — dadasnake expects raw fastq files
  and DADA2-based ASV delineation
- Data are not amplicon reads (e.g. shotgun metagenomes or genomes); dadasnake wraps
  DADA2 amplicon preprocessing only
- A one-off tool run is needed without workflow management — the configuration overhead
  (VARIABLE_CONFIG, sample table, YAML config) targets reproducible multi-step pipeline
  execution
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_3697
tools:
- name: dadasnake
  role: 1-command Snakemake workflow whose configuration files (YAML config, sample
    table, VARIABLE_CONFIG) are authored and validated in this skill
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: workflow engine that triggers multiple jobs from the single conda environment
    and reads the cluster config (e.g. config/slurm.config.yaml)
- name: conda
  role: environment manager used to install dadasnake, Snakemake, and the per-tool
    environments initialized at setup
- name: tmux
  role: terminal multiplexer used to keep the snakemake top instance running on the
    cluster frontend with the dadasnake_tmux wrapper
  repo: https://github.com/tmux/tmux/wiki
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:70ca09280101e7f24a10290189b9c9b941838926f834bab177344c4685846b87
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# snakemake-pipeline-configuration

## Summary

Configure and validate a dadasnake (Snakemake/DADA2) amplicon-processing run via a human-readable YAML configuration file, a tab-separated sample table, and installation-time VARIABLE_CONFIG settings so the workflow can execute reproducibly on a cluster or interactively.

## When to use

Use when raw paired-end FASTQ amplicon reads (e.g. ITS2 or 16S) must be processed with dadasnake and the pipeline needs to be installed, configured for a cluster or laptop, and dry-run validated before execution.

## When NOT to use

- Input is already an ASV/OTU or feature table — dadasnake expects raw fastq files and DADA2-based ASV delineation
- Data are not amplicon reads (e.g. shotgun metagenomes or genomes); dadasnake wraps DADA2 amplicon preprocessing only
- A one-off tool run is needed without workflow management — the configuration overhead (VARIABLE_CONFIG, sample table, YAML config) targets reproducible multi-step pipeline execution

## Inputs

- paired-end raw FASTQ files (r1/r2) of amplicon sequencing reads
- YAML configuration file (based on config/config.default.yaml) with pipeline step and database settings
- tab-separated sample table with library, r1_file, r2_file columns
- VARIABLE_CONFIG file for cluster/memory/scheduler settings (slurm or uge)

## Outputs

- validated, dry-run-checked dadasnake execution plan
- final ASV/OTU table with taxonomic classification
- standard hand-off formats: biom, phyloseq objects, FASTA, text/R data sets
- figures and tables reporting intermediary steps and statistics

## How to apply

Install dadasnake via conda after cloning the repository (git clone https://github.com/a-h-b/dadasnake.git), optionally installing Snakemake 6.9.1 via mamba into a conda/snakemake_env prefix. Adapt the VARIABLE_CONFIG file (tab-separated name/setting) to choose the cluster submission mode — dadasnake_allSubmit for submitting snakemake to the cluster, dadasnake_tmux to keep it on the frontend — and set SCHEDULER (slurm or uge), MAX_THREADS (default 50), NORMAL_MEM_EACH, and BIGMEM settings. Write a YAML config file pointing to the raw fastq files and reference databases (e.g. UNITE v8 for ITS, SILVA for 16S), and prepare a tab-separated sample table with library, r1_file, and r2_file column headers. Before the real run, validate both files with a dadasnake dryrun (-d); initialize the per-tool conda environments, then run the pipeline as a single command with Snakemake-triggered jobs.

## Related tools

- **dadasnake** (1-command Snakemake workflow whose configuration files (YAML config, sample table, VARIABLE_CONFIG) are authored and validated in this skill) — https://github.com/a-h-b/dadasnake
- **Snakemake** (workflow engine that triggers multiple jobs from the single conda environment and reads the cluster config (e.g. config/slurm.config.yaml))
- **conda** (environment manager used to install dadasnake, Snakemake, and the per-tool environments initialized at setup)
- **tmux** (terminal multiplexer used to keep the snakemake top instance running on the cluster frontend with the dadasnake_tmux wrapper) — https://github.com/tmux/tmux/wiki

## Examples

```
git clone https://github.com/a-h-b/dadasnake.git && cd dadasnake && dadasnake -d config.yaml sample_table.tsv
```

## Evaluation signals

- dadasnake dryrun (-d) completes without errors, validating the YAML config and the tab-separated sample table (library, r1_file, r2_file headers)
- VARIABLE_CONFIG is consistent with the target scheduler — SCHEDULER set to slurm or uge and the matching cluster config file (e.g. config/slurm.config.yaml) checked
- per-tool conda environments initialize successfully during the setup run
- the run produces an ASV table, taxonomy table, and standard hand-off formats (biom, fasta, tab-separated tables) as reported in figures and tables
- for the 19-species fungal mock community (PRJNA626434), exactly 19 fungal ASVs with expected intra-genus strain diversity (2 Aspergillus, 3 Fusarium) are recovered

## Limitations

- dadasnake requires Snakemake version >= 5.9.1 and has not been tested with previous versions
- LOCK_SETTINGS makes the workflow less flexible since all large data sets run with the maximum number of bigmem nodes configured at setup
- Some settings (SCHEDULER, SUBMIT_COMMAND, BIGMEM values) are cluster-specific and must be adjusted; other schedulers require contacting the developers via GitHub issues
- Snakemake job submission may need extra arguments such as --latency-wait=320 on slower file systems (SNAKEMAKE_EXTRA_ARGUMENTS)

## Evidence

- [abstract] user-friendly 1-command pipeline: "We present dadasnake, a user-friendly, 1-command Snakemake pipeline that wraps the preprocessing of sequencing reads and the delineation of exact sequence variants"
- [intro] human-readable configuration: "With a human-readable configuration file and a simple sample table, its steps are adjustable to a wide array of input data and requirements"
- [intro] cluster conda environment: "It is designed to run on a computing cluster using a single conda environment in multiple jobs triggered by Snakemake"
- [readme] VARIABLE_CONFIG setup: "Adjust the file VARIABLE_CONFIG to your requirements (have a tab between the variable name and your setting)"
- [methods] optional seqtk downsampling: "downsampling of reads per sample - using [seqtk](https://github.com/lh3/seqtk)"
