---
name: microbiome-benchmark-dataset-preparation
description: Use when preparing ground-truth mock-community inputs (amplicon FASTAs,
  genome labels, barcodes, real HiFi reads) to validate a PacBio HiFi amplicon simulator
  or compare simulated vs. observed ASV abundances.
when_to_use_negative:
- Input is already a simulated FASTQ/ASV count table — this skill stages real reference
  data, not downstream analysis of existing simulations.
- Target technology is Illumina short-read or ONT amplicon data — MHASS and its evaluation
  data are specific to PacBio HiFi amplicon sequencing.
- No genome-to-ASV mapping exists for the community — genome-aware abundance modeling
  requires the amplicon-genome label TSV.
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: MHASS
  role: Simulator run on the staged mock communities to produce synthetic HiFi amplicon
    datasets for the R2 comparison
  repo: https://github.com/rhowardstone/MHASS
- name: git
  role: Clones the MHASS_evaluation repository containing scripts and data paths
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: bash
  role: Runs scripts/setup.sh and scripts/1A_runner.sh to install dependencies and
    execute simulations
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: Python
  role: Computes Pearson R2 values comparing expected vs observed ASV abundances
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: AmpliconHunter
  role: Optionally generates the amplicons.fa input from a collection of genomes
  repo: https://github.com/rhowardstone/AmpliconHunter
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:f5f3e5c26abaf5e0481fd0275c045db9310097c7bb8809fc1bcdf209be77f4c3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - AmpliconHunter
  - MHASS
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# microbiome-benchmark-dataset-preparation

## Summary

Assembles the mock-community reference datasets (Zymo/Titan-1, ATCC 16S Sequel II 2.0, Phylotag) and evaluation scaffolding needed to benchmark HiFi amplicon simulators like MHASS against real PacBio data, by cloning the MHASS_evaluation repo, staging downloads, and wiring runner scripts.

## When to use

Use when preparing ground-truth mock-community inputs (amplicon FASTAs, genome labels, barcodes, real HiFi reads) to validate a PacBio HiFi amplicon simulator or compare simulated vs. observed ASV abundances.

## When NOT to use

- Input is already a simulated FASTQ/ASV count table — this skill stages real reference data, not downstream analysis of existing simulations.
- Target technology is Illumina short-read or ONT amplicon data — MHASS and its evaluation data are specific to PacBio HiFi amplicon sequencing.
- No genome-to-ASV mapping exists for the community — genome-aware abundance modeling requires the amplicon-genome label TSV.

## Inputs

- Amplicon reference FASTA (amplicons.fa) with ASV sequences
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- Barcode TSV with columns id, forward, reverse
- ATCC 16S Data Set Sequel II System 2.0 release (real PacBio HiFi reads)
- Phylotag dataset from JGI
- MHASS_evaluation data bundle (Google Drive zip)

## Outputs

- Simulated HiFi read FASTQ (combined_reads.fastq) per mock community
- counts.tsv (simulated ASV counts per sample) and counts_meta.tsv (library sizes)
- 1A_r2_results.csv with six Pearson R2 values (real vs simulated) for three datasets
- sample_barcode_map.tsv and sequence_file_mapping.tsv provenance files

## How to apply

Clone rhowardstone/MHASS_evaluation and install MHASS plus Python dependencies via 'bash scripts/setup.sh' (MHASS itself needs conda and samtools via install_dependencies.sh, activated with 'conda activate mhass'). Download and unzip the evaluation data bundle (Google Drive link in the README) into the same directory, then fetch the ATCC 16S Data Set Sequel II System 2.0 release from PacificBiosciences/DevNet and the Phylotag dataset from JGI PhyloTag. Update path references in the runner scripts to match the download locations, then run 'bash scripts/1A_runner.sh' to simulate the Zymo/Titan-1, ATCC/16S, and Phylotag/16S datasets. Parse simulator outputs (counts.tsv, counts_meta.tsv, sequence_file_mapping.tsv, combined_reads.fastq) and compute expected-vs-observed Pearson R2 at genome and ASV level with Python, assembling results into 1A_r2_results.csv.

## Related tools

- **MHASS** (Simulator run on the staged mock communities to produce synthetic HiFi amplicon datasets for the R2 comparison) — https://github.com/rhowardstone/MHASS
- **git** (Clones the MHASS_evaluation repository containing scripts and data paths) — https://github.com/rhowardstone/MHASS_evaluation
- **bash** (Runs scripts/setup.sh and scripts/1A_runner.sh to install dependencies and execute simulations) — https://github.com/rhowardstone/MHASS_evaluation
- **Python** (Computes Pearson R2 values comparing expected vs observed ASV abundances) — https://github.com/rhowardstone/MHASS_evaluation
- **AmpliconHunter** (Optionally generates the amplicons.fa input from a collection of genomes) — https://github.com/rhowardstone/AmpliconHunter

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git && cd MHASS_evaluation && bash scripts/setup.sh && bash scripts/1A_runner.sh
```

## Evaluation signals

- Computed R2 values match the reported correlations: 0.662/0.519 (Zymo/Titan-1 real/simulated), 0.956/0.999 (ATCC/16S), 0.794/0.944 (Phylotag/16S)
- Six R2 values (real and simulated for each of three datasets) are present in 1A_r2_results.csv
- Simulator output directory contains counts.tsv, counts_meta.tsv, sample_barcode_map.tsv, sequence_file_mapping.tsv, and combined_reads.fastq
- Runner scripts execute without path errors after updating references to downloaded data locations

## Limitations

- The ATCC dataset must be fetched from the archived PacificBiosciences/DevNet wiki and the Phylotag dataset from JGI, so external availability may change
- Runner scripts contain hard-coded path references that must be manually updated after download
- MHASS installation requires conda and samtools, which may not be available in all environments

## Evidence

- [methods] Clone the MHASS_evaluation repository with git and run scripts/setup.sh with bash to install MHASS and its Python dependencies: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [methods] Runner script execution step for the three mock communities: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:"
- [readme] README output file contract for each simulation run: [evidence span withheld — anchor status: external_doc]
- [methods] ATCC dataset download source: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [abstract] MHASS purpose as grounding for the benchmark: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
