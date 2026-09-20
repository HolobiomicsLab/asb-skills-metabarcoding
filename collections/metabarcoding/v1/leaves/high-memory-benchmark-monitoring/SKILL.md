---
name: high-memory-benchmark-monitoring
description: Use when you need to quantify runtime and memory requirements of a multi-sample
  amplicon workflow (e.g. >27,000 samples, billions of reads) on a bounded cluster
  allocation such as 50 cores.
when_to_use_negative:
- Input is already a processed feature/ASV table and no raw-read preprocessing will
  be run, so there is nothing to benchmark
- The dataset is small enough (e.g. a few samples on a laptop) that bigmem node requirements
  and cluster core limits are irrelevant
- You only need interactive exploratory analysis rather than a reproducible scalability
  measurement against a reference footprint (87 h / 190 GB / 50 cores)
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: dadasnake
  role: Snakemake pipeline being benchmarked; submitted in cluster mode with -c -n
    RUNNAME -r config.yaml and thread cap -t 50, monitored via RUNNAME stderr and
    log files until workflow.done appears
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: Dispatches the pipeline's jobs to at most 50 cores on the cluster, enabling
    efficient use of high-performance computing infrastructures during the benchmark
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Performs quality filtering/trimming, run-specific error-model estimation,
    denoising, and chimera removal — the most computationally demanding steps whose
    footprint is measured
- name: tmux
  role: 'Optional: keeps the Snakemake top instance running on the frontend if the
    benchmark is not fully submitted to the cluster'
  repo: https://github.com/tmux/tmux
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:03ff69e90aad8fb2081d5e78315e4c4b4c2f9085d8f47fbc6b0ef0d15fa09134
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  - tmux
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# high-memory-benchmark-monitoring

## Summary

A benchmarking skill for measuring wall-clock runtime and peak memory of large-scale amplicon processing: it reproduces the dadasnake/DADA2 pipeline on the Earth Microbiome Project scale (2.1 billion reads, >27,000 samples) on ≤50 CPU cores to characterize computational footprint (~87 real hours, >190 GB RAM).

## When to use

Use when you need to quantify runtime and memory requirements of a multi-sample amplicon workflow (e.g. >27,000 samples, billions of reads) on a bounded cluster allocation such as 50 cores.

## When NOT to use

- Input is already a processed feature/ASV table and no raw-read preprocessing will be run, so there is nothing to benchmark
- The dataset is small enough (e.g. a few samples on a laptop) that bigmem node requirements and cluster core limits are irrelevant
- You only need interactive exploratory analysis rather than a reproducible scalability measurement against a reference footprint (87 h / 190 GB / 50 cores)

## Inputs

- Raw amplicon sequencing FASTQ files (paired-end) for >27,000 Earth Microbiome Project samples (2.1 billion reads) placed in one raw_directory
- Tab-separated sample table with columns library, r1_file, r2_file, and run IDs
- YAML config file (dadasnake config.yaml) setting trunc_length=90 bp, paired processing, DADA2 denoising/chimera removal, and output directory

## Outputs

- Measured wall-clock runtime (expected ~87 real hours on ≤50 cores)
- Peak memory usage measurement (expected >190 GB during final ASV-table handling)
- Final ASV table for >700,000 ASVs handed off as biom format, R/phyloseq objects, FASTA, and tab-separated tables
- RUNNAME stderr file and per-rule Snakemake log files documenting progress

## How to apply

Place the raw amplicon FASTQ files in a single raw_directory and build a tab-separated sample table with library, r1_file, r2_file columns plus run IDs so run-specific DADA2 error models can be estimated; configure trunc_length to 90 bp with paired processing, DADA2-based denoising/chimera removal, and a unique output directory in a yaml config. First run 'dadasnake -d -r config.yaml' as a dryrun to validate paths, sample-table formatting, and step compatibility before submission. Then launch in cluster-submit mode with 'dadasnake -c -n RUNNAME -r config.yaml -t 50' so Snakemake dispatches jobs to at most 50 cores, monitoring progress via the RUNNAME stderr file and per-rule log files. When a 'workflow.done' file appears in the output directory, record the wall-clock duration (expected ~87 real hours) and peak memory usage (expected >190 GB, requiring bigmem nodes) during final ASV-table handling (>700,000 ASVs).

## Related tools

- **dadasnake** (Snakemake pipeline being benchmarked; submitted in cluster mode with -c -n RUNNAME -r config.yaml and thread cap -t 50, monitored via RUNNAME stderr and log files until workflow.done appears) — https://github.com/a-h-b/dadasnake
- **Snakemake** (Dispatches the pipeline's jobs to at most 50 cores on the cluster, enabling efficient use of high-performance computing infrastructures during the benchmark) — https://github.com/a-h-b/dadasnake
- **DADA2** (Performs quality filtering/trimming, run-specific error-model estimation, denoising, and chimera removal — the most computationally demanding steps whose footprint is measured)
- **tmux** (Optional: keeps the Snakemake top instance running on the frontend if the benchmark is not fully submitted to the cluster) — https://github.com/tmux/tmux

## Examples

```
dadasnake -c -n EMP_benchmark -r config.yaml -t 50
```

## Evaluation signals

- The dryrun ('dadasnake -d -r config.yaml') completes without errors, validating input paths, sample-table formatting, and step compatibility before the real submission
- The run finishes with a 'workflow.done' file in the output directory, indicating the Snakemake DAG completed under the -t 50 core cap
- Recorded wall-clock duration is on the order of ~87 real hours for the full 2.1-billion-read EMP dataset on ≤50 CPU cores
- Peak memory usage is on the order of >190 GB, consistent with processing the >700,000-ASV final table
- Final outputs exist in all expected hand-off formats: biom, phyloseq objects, FASTA, and tab-separated ASV tables

## Limitations

- The >190 GB peak memory requires bigmem/highmem compute nodes; on clusters without them (BIGMEM_CORES set to 0) such large datasets cannot be run with this configuration
- MAX_THREADS defaults to 50 if unset, so the benchmark footprint depends on cluster configuration in VARIABLE_CONFIG and config.default.yaml (normalMem, bigMem, bigCores)
- Wall-clock time depends on scheduler settings and file system latency (e.g. --latency-wait may need adjustment for slower file systems), so absolute runtimes may vary between clusters

## Evidence

- [other] EMP runtime/memory footprint finding: [evidence span withheld — anchor status: not_found]
- [other] Benchmark measurement instruction: [evidence span withheld — anchor status: not_found]
- [readme] Cluster execution design: "It is designed to run on a computing cluster using a single conda environment in multiple jobs triggered by Snakemake."
- [readme] Default thread cap: "MAX_THREADS - set this to the maximum number of cores you want to be using in a run. If you don't set this, the default will be 50."
- [abstract] HPC efficiency claim: "By use of Snakemake, dadasnake makes efficient use of high-performance computing infrastructures."
