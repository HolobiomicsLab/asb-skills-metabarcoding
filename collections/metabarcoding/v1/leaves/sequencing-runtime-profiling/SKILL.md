---
name: sequencing-runtime-profiling
description: Use when reproducing or extending the runtime/memory benchmark table
  for MHASS simulations, or when estimating computational cost before launching HiFi
  amplicon simulations at a given sample and read count.
when_to_use_negative:
- Your goal is assessing simulation fidelity (ASV clustering or taxonomic assignment
  accuracy) rather than computational resource usage
- You are profiling a real sequencing run rather than MHASS-generated synthetic PacBio
  HiFi amplicon datasets
- Only one dataset configuration is of interest and you do not need the linear scaling
  comparison across read counts
tools:
- name: MHASS
  role: Simulator whose runs are being timed and memory-profiled across the three
    validation datasets
  repo: https://github.com/rhowardstone/MHASS
- name: git
  role: Clones the MHASS_evaluation repository containing the runner and setup scripts
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: bash
  role: Executes scripts/setup.sh and scripts/1A_runner.sh to install dependencies
    and launch the three simulations in sequence
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: Python
  role: Optional resource monitoring (e.g. Python resource monitoring) to capture
    wall-clock time and peak memory, and aggregation into runtime_benchmark.csv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:fb963a0119d003502f8c4bc42d1cbda59125923814d75a82fe3130857ab4a2f0
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# sequencing-runtime-profiling

## Summary

Benchmark wall-clock runtime and peak memory of MHASS PacBio HiFi amplicon simulations across the three validation datasets (Zymo/Titan-1, ATCC/16S, Phylotag/16S), demonstrating that time and memory scale linearly with read count plus overhead from empirical pass-number sampling.

## When to use

Use when reproducing or extending the runtime/memory benchmark table for MHASS simulations, or when estimating computational cost before launching HiFi amplicon simulations at a given sample and read count.

## When NOT to use

- Your goal is assessing simulation fidelity (ASV clustering or taxonomic assignment accuracy) rather than computational resource usage
- You are profiling a real sequencing run rather than MHASS-generated synthetic PacBio HiFi amplicon datasets
- Only one dataset configuration is of interest and you do not need the linear scaling comparison across read counts

## Inputs

- MHASS_evaluation repository runner scripts (scripts/1A_runner.sh, scripts/setup.sh) with paths updated to local downloads
- Unzipped supplementary data file from the MHASS_evaluation release
- ATCC 16S Data Set Sequel II System 2.0 Release (PacificBiosciences DevNet)
- Phylotag dataset from JGI (PhyloTag portal)
- MHASS configuration files for the three validation datasets (Zymo/Titan-1, ATCC/16S, Phylotag/16S)

## Outputs

- runtime_benchmark.csv with wall-clock time and peak memory entries for all three dataset configurations
- Simulated HiFi amplicon FASTQ outputs (e.g. combined_reads.fastq) and counts tables (counts.tsv, counts_meta.tsv) per configuration

## How to apply

Clone the MHASS_evaluation repository and run `bash scripts/setup.sh` to install MHASS and its dependencies. Download and unzip the supplementary data file into the evaluation directory, then fetch the ATCC 16S dataset (PacificBiosciences DevNet Sequel II System 2.0 release) and the Phylotag dataset from JGI, and update the path references inside the runner scripts. Execute `bash scripts/1A_runner.sh` to run the MHASS simulations for the Zymo/Titan-1 (96 samples, ~2k reads/sample), ATCC/16S (192 samples, ~13k reads/sample), and Phylotag/16S (5 samples, ~23k reads/sample) configurations in sequence. Profile each run with a resource monitor such as `/usr/bin/time` or Python resource monitoring to capture wall-clock time and peak memory. Aggregate per-configuration measurements into runtime_benchmark.csv; expect runtimes of roughly 40 min (Zymo/Titan-1), 6.75 h (ATCC/16S), and 23 min (Phylotag/16S), scaling linearly with read count plus overhead from empirical pass-number sampling.

## Related tools

- **MHASS** (Simulator whose runs are being timed and memory-profiled across the three validation datasets) — https://github.com/rhowardstone/MHASS
- **git** (Clones the MHASS_evaluation repository containing the runner and setup scripts) — https://github.com/rhowardstone/MHASS_evaluation
- **bash** (Executes scripts/setup.sh and scripts/1A_runner.sh to install dependencies and launch the three simulations in sequence) — https://github.com/rhowardstone/MHASS_evaluation
- **Python** (Optional resource monitoring (e.g. Python resource monitoring) to capture wall-clock time and peak memory, and aggregation into runtime_benchmark.csv)

## Examples

```
/usr/bin/time -v bash scripts/1A_runner.sh
```

## Evaluation signals

- runtime_benchmark.csv exists and contains time and memory entries for all three configurations (Zymo/Titan-1, ATCC/16S, Phylotag/16S)
- Zymo/Titan-1 wall-clock runtime is approximately 40 min (96 samples, ~2k reads/sample)
- ATCC/16S wall-clock runtime is approximately 6.75 h (192 samples, ~13k reads/sample)
- Phylotag/16S wall-clock runtime is approximately 23 min (5 samples, ~23k reads/sample)
- Measured time and memory values are consistent with linear scaling in read count plus overhead from empirical pass-number sampling

## Limitations

- Timing is influenced by overhead from empirical pass-number sampling, so runtime is not purely proportional to read count
- The empirical np-distribution mode requires a --np-distribution TSV file; runs using the lognormal alternative (--np-distribution-type lognormal, mu=3.88, sigma=1.22, np-min=2, np-max=59) may not match the benchmarked timings
- Runner script paths must be manually updated to local download locations, and the MHASS default --threads is set to all CPUs, so measurements are hardware-dependent

## Evidence

- [results] MHASS simulation runtimes were 40 min for Zymo/Titan-1 (96 samples, ~2k reads/sample), 6.75 h for ATCC/16S (192 samples, ~13k reads/sample), and 23 min for Phylotag/16S (5 samples, ~23k reads/sample), with time and memory scaling linearly with read count plus overhead from empirical pass-number sampling.: [evidence span withheld — anchor status: not_found]
- [methods] Clone and setup of the evaluation repository: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [readme] Runner script execution instruction: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:"
- [readme] The benchmarked runner command: "bash scripts/1A_runner.sh"
- [methods] Dataset acquisition step for the ATCC validation data: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [readme] Supplementary data placement requirement: "Download this data file and unzip it to the same directory:"
