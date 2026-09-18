---
name: reference-amplicon-dataset-acquisition
description: Use when setting up the MHASS_evaluation workspace and you must obtain
  the real HiFi amplicon inputs (ATCC 16S, Phylotag, supplemental data archive) that
  parameterize the simulator.
when_to_use_negative:
- The reference datasets are already downloaded and the runner scripts already point
  to valid local paths
- You only need to run MHASS on your own amplicon FASTA/barcode inputs and do not
  intend to reproduce the Zymo, ATCC, or Phylotag validation benchmarks
tools:
- name: git
  role: Clones the MHASS_evaluation repository containing evaluation scripts and setup
    instructions
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: bash
  role: Runs scripts/setup.sh to install MHASS and its dependencies, and later runs
    the runner scripts after dataset paths are updated
- name: MHASS
  role: The simulator whose evaluation workspace and empirical pass-number distributions
    depend on the downloaded datasets
  repo: https://github.com/rhowardstone/MHASS
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:0e2086bc41f6a92e522a524d7f35e151a25bab234f4a50d98503bfc9769a8ecf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS_evaluation
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
yes_rate: 0.36363636363636365
n_claims: 11
---

# reference-amplicon-dataset-acquisition

## Summary

Acquires the three reference PacBio HiFi amplicon datasets (Zymo/Titan-1, ATCC 16S Sequel II 2.0, Phylotag 16S) plus the MHASS_evaluation supplementary data bundle so that MHASS simulations can be configured and benchmarked against real sequencing runs.

## When to use

Use when setting up the MHASS_evaluation workspace and you must obtain the real HiFi amplicon inputs (ATCC 16S, Phylotag, supplemental data archive) that parameterize the simulator.

## When NOT to use

- The reference datasets are already downloaded and the runner scripts already point to valid local paths
- You only need to run MHASS on your own amplicon FASTA/barcode inputs and do not intend to reproduce the Zymo, ATCC, or Phylotag validation benchmarks

## Inputs

- MHASS_evaluation git repository (github.com/rhowardstone/MHASS_evaluation)
- Supplementary data archive from Google Drive (file id 1F0I_pwu1e4gKK796CsgUZGGq2kQi3Bs0), unzipped into the evaluation directory
- ATCC 16S Data Set Sequel II System 2.0 Release from PacificBiosciences DevNet
- Phylotag 16S dataset from the JGI PhyloTag portal

## Outputs

- Locally staged Zymo/Titan-1, ATCC/16S, and Phylotag/16S reference datasets with runner-script path references updated
- Installed MHASS environment ready for simulation runs (e.g. conda environment 'mhass')

## How to apply

Clone the MHASS_evaluation repository and install MHASS and its dependencies via the setup script (bash scripts/setup.sh). Download and unzip the supplementary data file (hosted on Google Drive) into the same directory, then obtain the ATCC 16S dataset from the PacificBiosciences DevNet wiki (16S Data Set Sequel II System 2.0 Release) and the Phylotag dataset from the JGI PhyloTag portal. After the downloads are in place, update the path references inside the runner scripts (e.g. scripts/1A_runner.sh) to point at the local locations of the downloaded data before launching the simulations. Rationale: these empirical datasets supply the real amplicon, barcode, and pass-number characteristics that MHASS uses to construct realistic synthetic PacBio HiFi microbiome samples.

## Related tools

- **git** (Clones the MHASS_evaluation repository containing evaluation scripts and setup instructions) — https://github.com/rhowardstone/MHASS_evaluation
- **bash** (Runs scripts/setup.sh to install MHASS and its dependencies, and later runs the runner scripts after dataset paths are updated)
- **MHASS** (The simulator whose evaluation workspace and empirical pass-number distributions depend on the downloaded datasets) — https://github.com/rhowardstone/MHASS

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git; cd MHASS_evaluation; bash scripts/setup.sh
```

## Evaluation signals

- The supplementary data archive is unzipped in the same directory as the MHASS_evaluation clone
- The ATCC 16S Sequel II System 2.0 and Phylotag dataset files are present at the paths referenced in the runner scripts
- bash scripts/setup.sh completes and the mhass conda environment activates without dependency errors
- Running bash scripts/1A_runner.sh executes the Zymo/Titan-1, ATCC/16S, and Phylotag/16S simulation configurations without path-not-found failures

## Limitations

- The supplementary data file is hosted on Google Drive and the ATCC/Phylotag datasets on external portals, so availability and download locations may change
- Runner scripts contain hard-coded path references that must be manually edited to reflect the local download locations before execution

## Evidence

- [methods] Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [methods] And the Phylotag dataset: https://genome.jgi.doe.gov/portal/PhyloTag/PhyloTag.home.html: "And the Phylotag dataset: https://genome.jgi.doe.gov/portal/PhyloTag/PhyloTag.home.html"
- [readme] update the references in the runner scripts to reflect the location of your downloads: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:"
- [readme] Download this data file and unzip it to the same directory: "Download this data file and unzip it to the same directory"
