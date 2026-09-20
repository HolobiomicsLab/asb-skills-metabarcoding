---
name: wallclock-runtime-benchmarking
description: Use when measuring how long dadasnake takes to fully process a small
  amplicon dataset (e.g. 24-sample 16S V4, ~2.9 million reads) and whether runtime
  decreases with more cores.
when_to_use_negative:
- Input is already a processed ASV/OTU feature table rather than raw fastq files
- The goal is accuracy or taxonomic-composition benchmarking of the pipeline rather
  than runtime scaling
- Dataset is very large and not feasible to re-run from scratch twice to reset the
  output directory between core-count conditions
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: dadasnake
  role: 1-command Snakemake pipeline being benchmarked; run with -t to limit cpu count
    and -d for dry-run validation
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: Underlying workflow engine that makes efficient use of multi-core/HPC infrastructures
- name: conda
  role: Environment manager used to install snakemake and dadasnake dependencies
- name: tmux
  role: Optional terminal multiplexer for keeping the snakemake top instance running
    on the frontend
  repo: https://github.com/tmux/tmux
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:2c2f97edb922f3caf2a1ef2a27c919c5ea0ba40d9ca81c9dfd5c96bcc4bf239f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - Snakemake
  - conda
  - dadasnake
  - mothur
  - tmux
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# wallclock-runtime-benchmarking

## Summary

Benchmark the end-to-end wall-clock time of a dadasnake run on a small dataset while varying the core count, to quantify how Snakemake-based parallelization reduces processing time.

## When to use

Use when measuring how long dadasnake takes to fully process a small amplicon dataset (e.g. 24-sample 16S V4, ~2.9 million reads) and whether runtime decreases with more cores.

## When NOT to use

- Input is already a processed ASV/OTU feature table rather than raw fastq files
- The goal is accuracy or taxonomic-composition benchmarking of the pipeline rather than runtime scaling
- Dataset is very large and not feasible to re-run from scratch twice to reset the output directory between core-count conditions

## Inputs

- Raw 16S V4 fastq read files for 24 mock-community samples (NCBI SRA BioProject PRJNA517390, ~2.9 million reads)
- YAML configuration file with output directory, raw_directory, and sample_table path
- Tab-separated sample table with 'library', 'r1_file' (and 'r2_file') headers

## Outputs

- Wall-clock runtimes per core count (e.g. ~150 min on 1 core, ~100 min on 4 cores)
- workflow.done marker confirming full processing
- runtime_benchmark.csv with core counts, measured runtimes, and completion status

## How to apply

Obtain the raw 16S V4 fastq files (e.g. 24 mock-community samples from NCBI SRA BioProject PRJNA517390) into a single raw_directory. Write a YAML config file (output directory, raw_directory, sample_table path) plus a tab-separated sample table with 'library', 'r1_file' (and 'r2_file') headers. Validate inputs and permissions with a dry-run (dadasnake -d -r config.yaml). Then run the full workflow twice, once limited to 1 cpu and once to 4 cores via the -t option, recording wall-clock time from start until the workflow.done marker appears; the 24-sample dataset took ~150 min on one core and ~100 min on 4 cores. Compile core counts, runtimes, and completion status into a runtime_benchmark.csv.

## Related tools

- **dadasnake** (1-command Snakemake pipeline being benchmarked; run with -t to limit cpu count and -d for dry-run validation) — https://github.com/a-h-b/dadasnake
- **Snakemake** (Underlying workflow engine that makes efficient use of multi-core/HPC infrastructures)
- **conda** (Environment manager used to install snakemake and dadasnake dependencies)
- **tmux** (Optional terminal multiplexer for keeping the snakemake top instance running on the frontend) — https://github.com/tmux/tmux

## Examples

```
dadasnake -l -n RUN1 -t 1 -r config.yaml ; dadasnake -l -n RUN2 -t 4 -r config.yaml
```

## Evaluation signals

- workflow.done marker appears in the output directory after each run, confirming full completion
- Measured runtime on 4 cores is lower than on 1 core (~100 vs ~150 min for the 24-sample dataset)
- Dry-run (dadasnake -d -r config.yaml) reports correct input files, sample table, and write permissions before timing
- Both runs start from a reset output directory so timings are comparable
- runtime_benchmark.csv contains core count, wall-clock time, and completion status for each condition

## Limitations

- Users can override MAX_THREADS at runtime, so the effective cpu count must be controlled explicitly with -t
- Timings depend on the specific hardware/cluster (normal node RAM, scheduler) configured via VARIABLE_CONFIG; results are not directly portable
- The output directory must be reset between runs; partial reruns would not measure a full end-to-end wall-clock time

## Evidence

- [full_text] The 24-sample 16S V4 dataset was completely processed in a total wall clock time of 150 minutes on one core, and running time was reduced to 100 minutes when 4 cores were used.: [evidence span withheld — anchor status: not_found]
- [abstract] dadasnake is a single-command Snakemake pipeline making efficient use of computing infrastructure: "By use of Snakemake, dadasnake makes efficient use of high-performance computing infrastructures."
- [readme] MAX_THREADS sets the maximum number of cores used in a run, which users can override at runtime: "MAX_THREADS - set this to the maximum number of cores you want to be using in a run. If you don't set this, the default will be 50. Users can override this setting at runtime."
- [readme] Dry-run validates input files, sample table, and write permissions before executing the workflow: [evidence span withheld — anchor status: not_found]
