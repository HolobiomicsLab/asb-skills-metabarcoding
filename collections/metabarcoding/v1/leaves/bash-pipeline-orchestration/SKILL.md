---
name: bash-pipeline-orchestration
description: 'Use when reproducing the paper''s end-to-end evaluation pipeline from
  a cloned repository: bash scripts must be installed, rewired to local ATCC/Phylotag
  dataset downloads, executed in sequence, and profiled for wall-clock time and peak
  memory.'
when_to_use_negative:
- A single synthetic dataset is needed directly — invoke the mhass CLI (e.g., --amplicon-fasta,
  --num-samples, --np-distribution-type) instead of the full evaluation runner chain
- Downstream analysis of already-simulated data (e.g., ASV clustering or taxonomic
  assignment benchmarking) — this skill only orchestrates simulation setup, execution,
  and profiling
- Path references are already wired and prior simulation outputs exist — re-running
  setup.sh and the full runner would duplicate multi-hour simulations
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: MHASS
  role: Simulator launched sequentially by scripts/1A_runner.sh for the Zymo/Titan-1,
    ATCC/16S, and Phylotag/16S configurations; produces the FASTQ and count tables
    being benchmarked
  repo: https://github.com/rhowardstone/MHASS
- name: bash
  role: Runs scripts/setup.sh for installation and scripts/1A_runner.sh to sequence
    the three simulations
- name: git
  role: Clones the MHASS_evaluation repository that supplies the setup and runner
    scripts
- name: Python
  role: Implementation language of MHASS; also usable for resource monitoring to capture
    per-run time and memory
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:1630de168a396c0f0a98c42bb329809b0dcc8a9fd6c05228a13f4f82cf918681
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# bash-pipeline-orchestration

## Summary

Orchestrates the MHASS evaluation as chained bash scripts: install via scripts/setup.sh, repoint runner-script paths at local dataset downloads, then execute scripts/1A_runner.sh to simulate the Zymo/Titan-1, ATCC/16S, and Phylotag/16S configurations while profiling time and memory. This reproduces the paper's runtime_benchmark.csv measurements.

## When to use

Use when reproducing the paper's end-to-end evaluation pipeline from a cloned repository: bash scripts must be installed, rewired to local ATCC/Phylotag dataset downloads, executed in sequence, and profiled for wall-clock time and peak memory.

## When NOT to use

- A single synthetic dataset is needed directly — invoke the mhass CLI (e.g., --amplicon-fasta, --num-samples, --np-distribution-type) instead of the full evaluation runner chain
- Downstream analysis of already-simulated data (e.g., ASV clustering or taxonomic assignment benchmarking) — this skill only orchestrates simulation setup, execution, and profiling
- Path references are already wired and prior simulation outputs exist — re-running setup.sh and the full runner would duplicate multi-hour simulations

## Inputs

- Cloned MHASS_evaluation repository containing scripts/setup.sh and scripts/1A_runner.sh
- Supplementary evaluation data archive (Google Drive file 1F0I_pwu1e4gKK796CsgUZGGq2kQi3Bs0), unzipped into the evaluation directory
- ATCC 16S Data Set (Sequel II System 2.0 Release) from PacBio DevNet
- PhyloTag dataset from JGI
- Runner scripts with path placeholders to be updated to local download locations

## Outputs

- MHASS simulation outputs per configuration (combined_reads.fastq, counts.tsv, counts_meta.tsv, sample_barcode_map.tsv, sequence_file_mapping.tsv)
- Per-run wall-clock execution time and peak memory measurements
- runtime_benchmark.csv with time and memory entries for all three configurations

## How to apply

Clone rhowardstone/MHASS_evaluation with git and run `bash scripts/setup.sh` to install MHASS and its dependencies (installation requires conda and samtools), then activate the environment. Download the supplementary evaluation archive and unzip it into the same directory, then fetch the ATCC 16S Data Set (Sequel II System 2.0 Release) from PacBio DevNet and the PhyloTag dataset from JGI. Before execution, update the references in the runner scripts to reflect the location of your downloads — a required manual step because scripts/1A_runner.sh drives the Zymo/Titan-1, ATCC/16S, and Phylotag/16S MHASS configurations sequentially. Launch `bash scripts/1A_runner.sh`, profiling each run (e.g., /usr/bin/time or Python resource monitoring) to record wall-clock execution time and peak memory. Expect roughly 40 min (96 samples, ~2k reads/sample), 6.75 h (192 samples, ~13k reads/sample), and 23 min (5 samples, ~23k reads/sample) for the three configurations, since time and memory scale linearly with read count plus overhead from empirical pass-number sampling. Aggregate the per-configuration measurements into runtime_benchmark.csv and compare wall-clock values against these reported scales to confirm correct reproduction.

## Related tools

- **MHASS** (Simulator launched sequentially by scripts/1A_runner.sh for the Zymo/Titan-1, ATCC/16S, and Phylotag/16S configurations; produces the FASTQ and count tables being benchmarked) — https://github.com/rhowardstone/MHASS
- **bash** (Runs scripts/setup.sh for installation and scripts/1A_runner.sh to sequence the three simulations)
- **git** (Clones the MHASS_evaluation repository that supplies the setup and runner scripts)
- **Python** (Implementation language of MHASS; also usable for resource monitoring to capture per-run time and memory)

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git && cd MHASS_evaluation && bash scripts/setup.sh && bash scripts/1A_runner.sh
```

## Evaluation signals

- runtime_benchmark.csv exists and contains time and memory entries for all three configurations (Zymo/Titan-1, ATCC/16S, Phylotag/16S)
- Wall-clock values are consistent with the reported scale: ~40 min for 96 samples x ~2k reads/sample, ~6.75 h for 192 samples x ~13k reads/sample, ~23 min for 5 samples x ~23k reads/sample
- Each configuration's output directory contains combined_reads.fastq, counts.tsv, counts_meta.tsv, sample_barcode_map.tsv, and sequence_file_mapping.tsv
- Recorded time and memory trend linearly with per-sample read count, with extra overhead attributable to empirical pass-number sampling
- Runner scripts complete without path errors after references were updated to the download locations

## Limitations

- Runner-script path references must be manually updated to local download locations before 1A_runner.sh will run; stale paths abort the pipeline
- Benchmarks are hardware-dependent: the ATCC/16S configuration alone takes ~6.75 h, so the host must accommodate long sequential runs
- MHASS installation requires conda and samtools, and timing includes overhead from empirical pass-number sampling, so absolute values will differ from the paper's reported hardware measurements

## Evidence

- [methods] git clone https://github.com/rhowardstone/MHASS_evaluation.git: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [methods] update the references in the runner scripts to reflect the location of your downloads: "update the references in the runner scripts to reflect the location of your downloads"
- [methods] bash scripts/1A_runner.sh: "bash scripts/1A_runner.sh"
- [readme] Download this data file and unzip it to the same directory: "Download this data file and unzip it to the same directory"
- [readme] Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [readme] And the Phylotag dataset: https://genome.jgi.doe.gov/portal/PhyloTag/PhyloTag.home.html: "And the Phylotag dataset: https://genome.jgi.doe.gov/portal/PhyloTag/PhyloTag.home.html"
- [readme] Installation requires conda, samtools: [evidence span withheld — anchor status: external_doc]
