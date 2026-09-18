---
name: asv-abundance-comparison
description: Use when benchmarking a microbiome amplicon sequencing simulator against
  real PacBio HiFi mock community data by correlating expected (known mock composition)
  with observed ASV/genome abundances.
when_to_use_negative:
- Input is already an ASV count feature table with no expected mock composition or
  genome labels — genome-level correlation cannot be computed
- Data are short-read or shotgun metagenomic reads rather than PacBio HiFi amplicon
  sequencing, which the MHASS pass-number and error models target
- The goal is de novo ASV clustering or taxonomic assignment accuracy benchmarking
  rather than simulator abundance-fidelity evaluation
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3300
tools:
- name: MHASS
  role: Generates simulated PacBio HiFi amplicon datasets whose ASV abundances are
    compared against real mock community data
  repo: https://github.com/rhowardstone/MHASS
- name: MHASS_evaluation
  role: Provides runner scripts (1A_runner.sh, setup.sh) and data for the abundance
    correlation analysis
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: Python
  role: Computes Pearson R2 correlations and assembles results into 1A_r2_results.csv
- name: git
  role: Clones the MHASS_evaluation repository to obtain scripts and data
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: bash
  role: Runs scripts/setup.sh and scripts/1A_runner.sh to install dependencies and
    execute the simulation
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:95e803c86261d5a958bf6b760e933e242d32609753f8a531e458b2228173826b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - MHASS_evaluation
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# asv-abundance-comparison

## Summary

Compute and compare Pearson R2 correlations between expected and observed ASV abundances for real versus MHASS-simulated PacBio HiFi mock community datasets, at both genome and ASV level, to validate simulator realism.

## When to use

Use when benchmarking a microbiome amplicon sequencing simulator against real PacBio HiFi mock community data by correlating expected (known mock composition) with observed ASV/genome abundances.

## When NOT to use

- Input is already an ASV count feature table with no expected mock composition or genome labels — genome-level correlation cannot be computed
- Data are short-read or shotgun metagenomic reads rather than PacBio HiFi amplicon sequencing, which the MHASS pass-number and error models target
- The goal is de novo ASV clustering or taxonomic assignment accuracy benchmarking rather than simulator abundance-fidelity evaluation

## Inputs

- Amplicon reference FASTA (amplicons.fa) for each mock community
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- Real PacBio HiFi mock community sequencing data (ATCC 16S Data Set Sequel II System 2.0 release; Phylotag dataset; Zymo/Titan-1)
- Expected mock community composition (expected ASV abundances)
- MHASS simulator outputs (counts.tsv, counts_meta.tsv)

## Outputs

- 1A_r2_results.csv with six Pearson R2 values (real vs simulated) per dataset at genome and ASV level
- Per-dataset expected-vs-observed abundance correlation values (e.g. Zymo/Titan-1 R2=0.662 real vs 0.519 simulated; ATCC/16S R2=0.956 real vs 0.999 simulated; Phylotag/16S R2=0.794 vs 0.944)

## How to apply

Clone the MHASS_evaluation repository and run 'bash scripts/setup.sh' to install MHASS and its Python dependencies, then download and unzip the MHASS_evaluation data file, the ATCC 16S dataset (PacificBiosciences DevNet Sequel II System 2.0 release), and the Phylotag dataset into the evaluation directory. Update path references in the runner scripts to reflect the download locations, then run 'bash scripts/1A_runner.sh' to execute the MHASS simulator on the Zymo/Titan-1 (even abundances), ATCC/16S (staggered), and Phylotag/16S datasets. MHASS requires an amplicon FASTA and a TSV mapping amplicons to genomes (asvid<TAB>genomeid); for building the ASV FASTA from genomes the authors use AmpliconHunter. Parse simulator outputs (counts.tsv with simulated ASV counts per sample) and, with Python, compute Pearson R2 between expected and observed abundances at genome level and ASV level for each of the three datasets, assembling the six values into 1A_r2_results.csv. Rationale: matching the real-data correlations demonstrates that MHASS's genome-aware abundance modeling reproduces realistic abundance structure.

## Related tools

- **MHASS** (Generates simulated PacBio HiFi amplicon datasets whose ASV abundances are compared against real mock community data) — https://github.com/rhowardstone/MHASS
- **MHASS_evaluation** (Provides runner scripts (1A_runner.sh, setup.sh) and data for the abundance correlation analysis) — https://github.com/rhowardstone/MHASS_evaluation
- **Python** (Computes Pearson R2 correlations and assembles results into 1A_r2_results.csv)
- **git** (Clones the MHASS_evaluation repository to obtain scripts and data) — https://github.com/rhowardstone/MHASS_evaluation
- **bash** (Runs scripts/setup.sh and scripts/1A_runner.sh to install dependencies and execute the simulation)

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git; cd MHASS_evaluation; bash scripts/setup.sh; bash scripts/1A_runner.sh
```

## Evaluation signals

- Computed genome-level R2 values match the reported real-data correlations: 0.662 (Zymo/Titan-1), 0.956 (ATCC/16S), 0.794 (Phylotag/16S)
- Computed simulated-data ASV-level R2 values match: 0.519 (Zymo/Titan-1), 0.999 (ATCC/16S), 0.944 (Phylotag/16S)
- 1A_r2_results.csv contains all six R2 values in the correct real/simulated pairing per dataset
- counts.tsv from each MHASS run is non-empty and parseable with sample-level ASV counts consistent with the requested number of samples and reads
- Runner scripts complete without path errors after references were updated to the downloaded data locations

## Limitations

- Correlations are dataset-dependent: the simulated ASV-level R2 for Zymo/Titan-1 (0.519) is lower than the real value (0.662), so perfect agreement is not expected for even-abundance communities
- Running the evaluation requires external data downloads (Google Drive data file, PacificBiosciences DevNet ATCC dataset, JGI Phylotag portal) that may move or require registration
- MHASS installation requires conda and samtools, and the simulator defaults (e.g. subread accuracy 0.65, difference ratio 6:55:39 for PacBio RS II) must be chosen appropriately per platform

## Evidence

- [results] Expected and observed ASV abundance correlations were: Zymo/Titan-1 (even abundances): R2=0.662 (real), R2=0.519 (simulated); ATCC/16S (staggered): R2=0.956 (real), R2=0.999 (simulated).: [evidence span withheld — anchor status: not_found]
- [methods] git clone https://github.com/rhowardstone/MHASS_evaluation.git and run scripts/setup.sh with bash: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [readme] update the references in the runner scripts to reflect the location of your downloads, and run: bash scripts/1A_runner.sh: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:

bash
bash scripts/1A_runner.sh"
- [abstract] MHASS creates realistic synthetic PacBio HiFi amplicon sequencing datasets by integrating genome-aware abundance modeling: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
