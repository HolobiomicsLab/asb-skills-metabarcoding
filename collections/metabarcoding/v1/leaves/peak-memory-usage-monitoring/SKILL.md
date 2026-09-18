---
name: peak-memory-usage-monitoring
description: 'Use when benchmarking simulation or pipeline performance: you have launched
  per-configuration simulation runs and need to capture wall-clock time and peak RSS
  per run to build a runtime/memory benchmark table.'
when_to_use_negative:
- You only need MHASS simulation output data (counts.tsv, combined_reads.fastq) and
  are not evaluating runtime or memory characteristics
- The workloads are short single-shot commands whose overhead dominates measurement
  — benchmarking is meant for full dataset-scale simulation runs
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MHASS
  role: Simulator whose per-configuration runs are being profiled for time and peak
    memory
  repo: https://github.com/rhowardstone/MHASS
- name: bash
  role: Used to execute the runner script (1A_runner.sh) and the profiling wrapper
    for each simulation
- name: Python
  role: Optional resource-monitoring implementation for capturing wall-clock time
    and peak memory; MHASS itself is implemented in Python
- name: git
  role: Clones the MHASS_evaluation repository containing the runner scripts to profile
  repo: https://github.com/rhowardstone/MHASS_evaluation.git
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:a629a04ba7ccb6854d8195d186f0d6d07192518450000e1b31c6ed4ca6afee6e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  - git
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# Peak memory usage monitoring

## Summary

Profiles wall-clock execution time and peak memory of MHASS PacBio HiFi amplicon simulations across the three validation datasets (Zymo/Titan-1, ATCC/16S, Phylotag/16S) so runtime and memory scaling with read count can be benchmarked.

## When to use

Use when benchmarking simulation or pipeline performance: you have launched per-configuration simulation runs and need to capture wall-clock time and peak RSS per run to build a runtime/memory benchmark table.

## When NOT to use

- You only need MHASS simulation output data (counts.tsv, combined_reads.fastq) and are not evaluating runtime or memory characteristics
- The workloads are short single-shot commands whose overhead dominates measurement — benchmarking is meant for full dataset-scale simulation runs

## Inputs

- MHASS simulation runs for the three validation configurations (Zymo/Titan-1: 96 samples ~2k reads/sample; ATCC/16S: 192 samples ~13k reads/sample; Phylotag/16S: 5 samples ~23k reads/sample)
- Runner scripts (scripts/1A_runner.sh) configured with paths to downloaded Zymo, ATCC, and Phylotag datasets

## Outputs

- runtime_benchmark.csv containing per-configuration wall-clock time and peak memory entries for all three datasets
- Per-run profiler logs of elapsed time and peak memory (e.g. from /usr/bin/time or Python resource monitoring)

## How to apply

Launch each MHASS simulation configuration in sequence via the evaluation runner (bash scripts/1A_runner.sh) after cloning MHASS_evaluation, running scripts/setup.sh, and pointing the runner scripts at the downloaded Zymo, ATCC 16S (Sequel II System 2.0 release), and Phylotag datasets. Wrap or monitor each run with a profiler such as /usr/bin/time or Python resource monitoring to record elapsed wall-clock time and peak memory usage. Expect runtimes to scale linearly with per-sample read count (e.g. 96 samples at ~2k reads/sample vs 192 samples at ~13k reads/sample vs 5 samples at ~23k reads/sample), with additional overhead from MHASS's empirical pass-number (--np-distribution-type empirical) sampling. Aggregate per-configuration measurements into a summary table written to runtime_benchmark.csv, with one row per dataset configuration containing time and memory entries.

## Related tools

- **MHASS** (Simulator whose per-configuration runs are being profiled for time and peak memory) — https://github.com/rhowardstone/MHASS
- **bash** (Used to execute the runner script (1A_runner.sh) and the profiling wrapper for each simulation)
- **Python** (Optional resource-monitoring implementation for capturing wall-clock time and peak memory; MHASS itself is implemented in Python)
- **git** (Clones the MHASS_evaluation repository containing the runner scripts to profile) — https://github.com/rhowardstone/MHASS_evaluation.git

## Examples

```
/usr/bin/time -v bash scripts/1A_runner.sh 2> runtime_profile.log
```

## Evaluation signals

- runtime_benchmark.csv exists and contains both time and memory entries for all three configurations (Zymo/Titan-1, ATCC/16S, Phylotag/16S)
- Recorded wall-clock values are consistent with the reported benchmarks: ~40 min for Zymo/Titan-1, ~6.75 h for ATCC/16S, ~23 min for Phylotag/16S
- Time and memory measurements increase roughly linearly with total read count across the three dataset scales
- Peak memory values are positive, plausible for the host machine, and captured for every completed run (no missing entries)
- Profiling spans the entire simulation run including overhead from empirical pass-number sampling, not just partial phases

## Limitations

- Memory and time scaling includes overhead from MHASS's empirical pass-number sampling, so measurements are not a pure function of read count
- Reported benchmark values (40 min, 6.75 h, 23 min) depend on the original measurement hardware; reproductions on other machines may differ
- Accurate profiling requires runner-script path references to be correctly updated to the downloaded dataset locations before launching runs

## Evidence

- [methods] Clone the MHASS_evaluation repository with git and run the bash setup script (scripts/setup.sh) to install MHASS and its dependencies.: [evidence span withheld — anchor status: not_found]
- [readme] bash scripts/1A_runner.sh: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:"
- [methods] update the references in the runner scripts to reflect the location of your downloads: "update the references in the runner scripts to reflect the location of your downloads"
