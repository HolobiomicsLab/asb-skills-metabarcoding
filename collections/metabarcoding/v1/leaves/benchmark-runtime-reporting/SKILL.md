---
name: benchmark-runtime-reporting
description: Use when evaluating the computational cost of a simulator or pipeline
  and you must profile wall-clock time and peak memory per configuration, then aggregate
  measurements into a benchmark table.
when_to_use_negative:
- When the goal is to assess simulation accuracy (e.g. ASV clustering or taxonomic
  assignment fidelity) rather than computational cost
- When inputs are raw sequencing reads or FASTQ files to be simulated — this skill
  profiles simulator runs, it does not generate data
- When runs cannot be executed on fixed, comparable hardware, since scaling claims
  (linear with read count) depend on a consistent measurement environment
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MHASS
  role: Simulator being benchmarked; produces the PacBio HiFi amplicon datasets whose
    runtimes are profiled
  repo: https://github.com/rhowardstone/MHASS
- name: bash
  role: Runs scripts/setup.sh and scripts/1A_runner.sh to install and launch the three
    simulation configurations
- name: git
  role: Clones the MHASS_evaluation repository containing the runner scripts
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: Python
  role: MHASS implementation and vehicle for resource monitoring / aggregation of
    timing and memory measurements
  repo: https://github.com/rhowardstone/MHASS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:884f4ce23ba821ac487e5f91437b514e550a332a7ecf44cebbde3ed402af4ea5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# benchmark-runtime-reporting

## Summary

Reproduce and report the wall-clock runtime and peak-memory characteristics of MHASS simulations across the three validation datasets (Zymo/Titan-1, ATCC/16S, Phylotag/16S), profiling each run and aggregating results into a runtime_benchmark.csv summary table.

## When to use

Use when evaluating the computational cost of a simulator or pipeline and you must profile wall-clock time and peak memory per configuration, then aggregate measurements into a benchmark table.

## When NOT to use

- When the goal is to assess simulation accuracy (e.g. ASV clustering or taxonomic assignment fidelity) rather than computational cost
- When inputs are raw sequencing reads or FASTQ files to be simulated — this skill profiles simulator runs, it does not generate data
- When runs cannot be executed on fixed, comparable hardware, since scaling claims (linear with read count) depend on a consistent measurement environment

## Inputs

- MHASS_evaluation repository (github.com/rhowardstone/MHASS_evaluation) with scripts/setup.sh and scripts/1A_runner.sh
- Supplementary data file (Google Drive zip) unzipped into the evaluation directory
- ATCC 16S Data Set Sequel II System 2.0 Release from PacificBiosciences DevNet
- Phylotag dataset from JGI (genome.jgi.doe.gov PhyloTag portal)
- MHASS simulation configurations for Zymo/Titan-1, ATCC/16S, and Phylotag/16S

## Outputs

- runtime_benchmark.csv containing wall-clock time and peak memory entries for all three configurations
- Per-run profiling records (execution time and peak memory) for each MHASS simulation

## How to apply

Clone rhowardstone/MHASS_evaluation and run scripts/setup.sh to install MHASS and dependencies, then download the supplementary data file, the ATCC 16S dataset (PacificBiosciences DevNet Sequel II System 2.0 release), and the Phylotag dataset from JGI. Update path references inside the runner scripts, then launch the simulations for the three configurations in sequence with 'bash scripts/1A_runner.sh'. Profile each run with /usr/bin/time or Python resource monitoring to capture wall-clock execution time and peak memory, keeping hardware fixed so values are comparable. Report runtimes relative to dataset scale: 40 min for Zymo/Titan-1 (96 samples, ~2k reads/sample), 6.75 h for ATCC/16S (192 samples, ~13k reads/sample), and 23 min for Phylotag/16S (~5 samples, ~23k reads/sample), noting that time and memory scale linearly with read count plus overhead from empirical pass-number sampling. Aggregate the per-configuration measurements into runtime_benchmark.csv with time and memory entries for all three configurations.

## Related tools

- **MHASS** (Simulator being benchmarked; produces the PacBio HiFi amplicon datasets whose runtimes are profiled) — https://github.com/rhowardstone/MHASS
- **bash** (Runs scripts/setup.sh and scripts/1A_runner.sh to install and launch the three simulation configurations)
- **git** (Clones the MHASS_evaluation repository containing the runner scripts) — https://github.com/rhowardstone/MHASS_evaluation
- **Python** (MHASS implementation and vehicle for resource monitoring / aggregation of timing and memory measurements) — https://github.com/rhowardstone/MHASS

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git; cd MHASS_evaluation; bash scripts/setup.sh; bash scripts/1A_runner.sh
```

## Evaluation signals

- runtime_benchmark.csv exists and contains time and memory entries for all three configurations (Zymo/Titan-1, ATCC/16S, Phylotag/16S)
- Wall-clock values are consistent with the reported benchmark runtimes: 40 min (Zymo/Titan-1), 6.75 h (ATCC/16S), 23 min (Phylotag/16S)
- Reported time and memory scale linearly with read count, with additional overhead from empirical pass-number sampling
- Each simulation completed all samples in its configuration (e.g. 96 samples for Zymo/Titan-1, 192 for ATCC/16S) before profiling values were recorded
- Profiling was performed on consistent hardware so cross-configuration comparisons are valid

## Limitations

- Runtimes depend on hardware and thread count (MHASS defaults to 190 threads or all CPUs), so absolute values are not portable across machines
- Time and memory include overhead from empirical pass-number sampling, not just per-read simulation cost
- The default np distribution is empirical and requires a --np-distribution TSV file when --np-distribution-type is empirical

## Evidence

- [methods] git clone https://github.com/rhowardstone/MHASS_evaluation.git: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [readme] Runner script invocation: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:

bash
bash scripts/1A_runner.sh"
- [readme] Number of threads parameter: [evidence span withheld — anchor status: external_doc]
- [abstract] MHASS purpose statement: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
