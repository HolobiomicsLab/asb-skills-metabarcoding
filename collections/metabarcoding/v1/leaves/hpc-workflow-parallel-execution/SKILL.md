---
name: hpc-workflow-parallel-execution
description: Use when estimating or optimizing wall-clock runtime of a multi-sample
  dadasnake/DADA2 amplicon run on a cluster, or when deciding how many cores (-t)
  to allocate for a given dataset size.
when_to_use_negative:
- You only need ASV results and have no runtime or resource-budgeting question — run
  dadasnake once with a sensible thread cap instead of benchmarking.
- Your workflow is a single-sample or very small run where per-sample Snakemake parallelization
  cannot yield meaningful speedup and the overhead of cluster submission outweighs
  gains.
- Input is already a processed feature/OTU table — no amplicon preprocessing stages
  remain to parallelize.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0769
- http://edamontology.org/topic_0695
tools:
- name: dadasnake
  role: Pipeline whose thread cap (-t) and VARIABLE_CONFIG MAX_THREADS/NORMAL_MEM_EACH/SCHEDULER
    settings control parallel execution on the cluster
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: Workflow engine that makes efficient use of HPC infrastructures by triggering
    multiple per-sample jobs in parallel
  repo: https://snakemake.readthedocs.io/en/stable/
- name: conda
  role: Provides the single conda environment (snakemake installed via mamba, snakemake=6.9.1)
    in which all parallel jobs run
  repo: https://www.anaconda.com/
- name: tmux
  role: Keeps the Snakemake top instance running on the cluster frontend when jobs
    are not fully submitted to the scheduler
  repo: https://github.com/tmux/tmux/wiki
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:a1f8ee9d9cbeba95d2d6368b286049c26dd704dd8c4679f987ff673816774dc4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  - tmux
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0769
  - http://edamontology.org/topic_0695
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# hpc-workflow-parallel-execution

## Summary

Benchmark and exploit Snakemake-based parallelization of the dadasnake amplicon pipeline on HPC resources, using the thread cap (-t) to scale runtime with core count, as demonstrated on the 267-sample, 46.8-million-read ITS1 dataset (PRJNA272747) where 4 cores took just under 4 hours and 15 cores halved the runtime.

## When to use

Use when estimating or optimizing wall-clock runtime of a multi-sample dadasnake/DADA2 amplicon run on a cluster, or when deciding how many cores (-t) to allocate for a given dataset size.

## When NOT to use

- You only need ASV results and have no runtime or resource-budgeting question — run dadasnake once with a sensible thread cap instead of benchmarking.
- Your workflow is a single-sample or very small run where per-sample Snakemake parallelization cannot yield meaningful speedup and the overhead of cluster submission outweighs gains.
- Input is already a processed feature/OTU table — no amplicon preprocessing stages remain to parallelize.

## Inputs

- Tab-separated sample table with 'library', 'r1_file', 'r2_file' columns listing paired-end ITS1 fastq files (267 samples, 46.8 million reads, BioProject PRJNA272747)
- dadasnake config.yaml (paired=true, ITSx region=ITS1, mothur fungal ITS classification, pair_merging min_overlap >= 20)
- VARIABLE_CONFIG settings (MAX_THREADS, NORMAL_MEM_EACH, SCHEDULER)
- Compute allocation (e.g. four or fifteen 8 GB cores)

## Outputs

- Complete dadasnake output directory including 'workflow.done' marker file
- runtime_benchmark.tsv with columns (threads, wall-clock hours) for each thread cap tested
- Runtime scaling comparison (e.g. ~4 h at -t 4 vs ~2 h at -t 15)

## How to apply

First run a dryrun (dadasnake -d) to validate the sample table (columns 'library', 'r1_file', 'r2_file'), config.yaml (paired=true, ITSx region=ITS1, mothur classification against a UNITE-formatted fungal ITS database, pair_merging min_overlap >= 20 bp), and input fastq paths before committing compute. Set MAX_THREADS in VARIABLE_CONFIG to the core budget for the run (default 50; overridable at runtime) and set NORMAL_MEM_EACH to the RAM per core (e.g. 8G); optionally run the Snakemake top instance via tmux on the frontend or submit it to slurm/uge with SCHEDULER. Execute dadasnake with the thread cap (e.g. -t 4 on four 8 GB cores), record wall-clock time, then re-execute the identical workflow with a larger cap (e.g. -t 15) to measure the parallel speedup obtained from Snakemake-triggered per-sample jobs. Rationale: Snakemake parallelizes the per-sample stages (cutadapt primer removal, DADA2 quality filtering/denoising/ASV inference, merging, chimera removal, ITSx, mothur), so scaling the thread cap reduces runtime roughly proportionally for medium-sized datasets.

## Related tools

- **dadasnake** (Pipeline whose thread cap (-t) and VARIABLE_CONFIG MAX_THREADS/NORMAL_MEM_EACH/SCHEDULER settings control parallel execution on the cluster) — https://github.com/a-h-b/dadasnake
- **Snakemake** (Workflow engine that makes efficient use of HPC infrastructures by triggering multiple per-sample jobs in parallel) — https://snakemake.readthedocs.io/en/stable/
- **conda** (Provides the single conda environment (snakemake installed via mamba, snakemake=6.9.1) in which all parallel jobs run) — https://www.anaconda.com/
- **tmux** (Keeps the Snakemake top instance running on the cluster frontend when jobs are not fully submitted to the scheduler) — https://github.com/tmux/tmux/wiki

## Examples

```
dadasnake -t 4 && dadasnake -t 15  # identical config, record wall-clock time of each run
```

## Evaluation signals

- 'workflow.done' marker file present in the output directory after each run, confirming the workflow completed.
- Recorded wall-clock time on four 8 GB cores is just under 4 hours for the 267-sample, 46.8-million-read ITS1 dataset (Fig. 2c).
- Re-running with -t 15 yields approximately a 2-fold runtime reduction (Fig. 2d), recorded in runtime_benchmark.tsv.
- Dryrun (dadasnake -d) passes without errors, validating sample table formatting, config.yaml, and fastq input paths before compute is committed.
- All expected pipeline stages (cutadapt, DADA2, merging, chimera removal, ITSx, mothur) produced outputs, confirming no stage was skipped during parallel execution.

## Limitations

- Parallel speedup is not linear: 15 cores gave only ~2x speedup over 4 cores on the medium ITS1 dataset, so scaling gains diminish with additional cores.
- Cluster-specific configuration is required (SCHEDULER limited to slurm or uge; BIGMEM cores/nodes must be configured separately; BIND_JOBS_TO_MAIN needs the correct NODENAME_VAR such as SLURMD_NODENAME).
- Slow file systems may need extra Snakemake arguments (e.g. --latency-wait=320) via SNAKEMAKE_EXTRA_ARGUMENTS.
- dadasnake requires Snakemake version >= 5.9.1 and has not been tested with earlier versions.

## Evidence

- [abstract] Snakemake parallelization on HPC: "By use of Snakemake, dadasnake makes efficient use of high-performance computing infrastructures."
- [readme] Thread cap configuration: "MAX_THREADS - set this to the maximum number of cores you want to be using in a run. If you don't set this, the default will be 50. Users can override this setting at runtime."
- [readme] Memory per core setting: "NORMAL_MEM_EACH - set the size of the RAM of one core of your normal copute nodes (e.g. 8G)."
- [results] Runtime benchmark finding: [evidence span withheld — anchor status: not_found]
- [other] Completion validation: [evidence span withheld — anchor status: not_found]
