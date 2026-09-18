---
name: mock-community-simulation-benchmarking
description: Use when validating that simulated PacBio HiFi amplicon reads reproduce
  per-genome error profiles observed in real mock-community sequencing runs, e.g.
  before benchmarking ASV clustering or taxonomic assignment pipelines.
when_to_use_negative:
- Input is already a feature/ASV count table rather than raw simulated reads — error-profile
  comparison requires per-read sequences
- Reads are short-read (e.g. Illumina) rather than PacBio HiFi amplicon data, since
  MHASS models empirically derived HiFi pass-number and error distributions
- The goal is downstream benchmarking of ASV clustering or taxonomic assignment on
  an already-validated simulator rather than validation of the simulator's error model
  itself
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MHASS
  role: Generates the synthetic PacBio HiFi amplicon reads whose per-genome edit distance
    distributions are compared to real-data distributions
  repo: https://github.com/rhowardstone/MHASS
- name: MHASS_evaluation
  role: Provides evaluation scripts (setup.sh, 1A_runner.sh) and data for reproducing
    all analyses in the MHASS evaluation
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: Python
  role: Hosts the pairwise sequence alignment routine used to compute Levenshtein
    edit distances between simulated reads and reference ASVs
- name: bash
  role: Runs the setup and runner scripts that execute the simulation pipeline
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:2216a71d81c4c07e2fbb0e063e3a4a2a0fdb1e969ca92345d9bf927c49b53b48
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - MHASS_evaluation
  - Python
  - bash
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# mock-community-simulation-benchmarking

## Summary

Benchmark a microbiome HiFi amplicon simulator by comparing per-genome edit distance distributions of simulated PacBio HiFi reads against real sequencing data from mock communities (Zymo/Titan-1, ATCC/16S, Phylotag/16S). This validates that simulated read error profiles mirror empirical ones before using the simulator for downstream benchmarking.

## When to use

Use when validating that simulated PacBio HiFi amplicon reads reproduce per-genome error profiles observed in real mock-community sequencing runs, e.g. before benchmarking ASV clustering or taxonomic assignment pipelines.

## When NOT to use

- Input is already a feature/ASV count table rather than raw simulated reads — error-profile comparison requires per-read sequences
- Reads are short-read (e.g. Illumina) rather than PacBio HiFi amplicon data, since MHASS models empirically derived HiFi pass-number and error distributions
- The goal is downstream benchmarking of ASV clustering or taxonomic assignment on an already-validated simulator rather than validation of the simulator's error model itself

## Inputs

- MHASS-simulated HiFi amplicon reads (combined_reads.fastq) for Zymo/Titan-1, ATCC/16S, and Phylotag/16S mock communities
- Reference ASV sequences (amplicons.fa) for each mock community
- Real sequencing data from the ATCC 16S Data Set (Sequel II System 2.0 Release) and the JGI Phylotag dataset
- Amplicon-to-genome label TSV mapping each ASV to a genome ID

## Outputs

- Per-genome minimum edit distance distributions for each mock community
- edit_distance_distributions.csv (per-genome distribution summary table)

## How to apply

Clone the MHASS_evaluation repository, run 'bash scripts/setup.sh', download the data bundle plus the ATCC dataset (PacificBiosciences DevNet 16S Data Set Sequel II System 2.0 Release) and the Phylotag dataset from JGI, then run 'bash scripts/1A_runner.sh' to generate MHASS-simulated reads. For each simulated read, compute the global (Levenshtein) edit distance against every candidate reference ASV using a pairwise sequence alignment routine in Python, taking the minimum as the nearest-reference error. Aggregate per-read minimum edit distances into per-genome distributions for each mock community. Compare each simulated per-genome distribution against the real-data distributions reported in Fig. 1, checking for the expected patterns: bimodal for Zymo/Titan-1, unimodal for ATCC/16S, and realistic low-abundance behavior for Phylotag/16S genomes.

## Related tools

- **MHASS** (Generates the synthetic PacBio HiFi amplicon reads whose per-genome edit distance distributions are compared to real-data distributions) — https://github.com/rhowardstone/MHASS
- **MHASS_evaluation** (Provides evaluation scripts (setup.sh, 1A_runner.sh) and data for reproducing all analyses in the MHASS evaluation) — https://github.com/rhowardstone/MHASS_evaluation
- **Python** (Hosts the pairwise sequence alignment routine used to compute Levenshtein edit distances between simulated reads and reference ASVs)
- **bash** (Runs the setup and runner scripts that execute the simulation pipeline)

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git && cd MHASS_evaluation && bash scripts/setup.sh && bash scripts/1A_runner.sh
```

## Evaluation signals

- Simulated per-genome edit distance distributions for Zymo/Titan-1 are bimodal, matching the real-data pattern in Fig. 1
- Simulated per-genome edit distance distributions for ATCC/16S are unimodal, matching real data
- Phylotag/16S genomes show realistic low-abundance behavior in the simulated distributions
- Every simulated read has a well-defined minimum edit distance against at least one candidate reference ASV
- edit_distance_distributions.csv is produced with per-genome summary rows for all three mock communities

## Limitations

- Edit distance is global (Levenshtein); structural differences beyond simple substitution/insertion/deletion error are not captured
- Validation depends on external datasets (ATCC DevNet, JGI Phylotag) that must be downloaded separately and referenced correctly in runner scripts
- Real-data reference distributions come from Fig. 1 of the paper; agreement is qualitative (shape matching) rather than a formal statistical test

## Evidence

- [abstract] Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [results] Across all three mock communities, MHASS generated reads with edit distance distributions that closely mirrored those observed in real sequencing data.: "Across all three mock communities, MHASS generated reads with edit distance distributions that closely mirrored those observed in real sequencing data."
- [readme] Evaluation scripts and data for all analyses in our evaluation of MHASS: "Evaluation scripts and data for all analyses in our evaluation of MHASS"
- [methods] Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
