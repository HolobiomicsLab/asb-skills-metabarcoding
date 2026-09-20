---
name: per-position-indel-error-rate-quantification
description: Use when aligned PacBio HiFi amplicon reads show an unexplained secondary
  peak in the edit-distance distribution and you need to test whether the excess errors
  are insertion-specific and position-dependent (e.g., rising near one alignment position)
  rather than uniformly distributed.
when_to_use_negative:
- Reads have not yet been aligned to a reference sequence — per-position error proportions
  cannot be tabulated from unaligned FASTQ alone
- The question is an aggregate error rate or overall read accuracy, not a position-specific
  insertion/substitution/deletion profile
- Data are short-read (e.g., Illumina) rather than PacBio HiFi amplicon data, where
  the terminal insertion profile of this pipeline does not apply
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: MHASS
  role: Generates the simulated PacBio HiFi amplicon reads whose per-position error
    profile is compared against real reads (error model controlled by the PBSIM difference-ratio
    substitution:insertion:deletion parameter)
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Tabulates per-alignment-position insertion/substitution/deletion counts and
    plots the real vs. simulated error-rate traces with matplotlib
- name: bash
  role: Runs the MHASS_evaluation runner scripts (e.g., 1A_runner.sh) that produce
    the datasets fed into this analysis
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:21c5240f64e17bd19a1cea70e4820d978be203d5684130f5adae9df343d8dfdd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# per-position-indel-error-rate-quantification

## Summary

Quantify insertion, substitution, and deletion error proportions at each alignment position for real and MHASS-simulated PacBio HiFi amplicon reads, to determine whether a secondary edit-distance peak arises from position-specific (terminal) insertion errors. Applied by overlaying real vs. simulated per-position error traces and using 16S-only datasets (ATCC, Phylotag) as negative controls.

## When to use

Use when aligned PacBio HiFi amplicon reads show an unexplained secondary peak in the edit-distance distribution and you need to test whether the excess errors are insertion-specific and position-dependent (e.g., rising near one alignment position) rather than uniformly distributed.

## When NOT to use

- Reads have not yet been aligned to a reference sequence — per-position error proportions cannot be tabulated from unaligned FASTQ alone
- The question is an aggregate error rate or overall read accuracy, not a position-specific insertion/substitution/deletion profile
- Data are short-read (e.g., Illumina) rather than PacBio HiFi amplicon data, where the terminal insertion profile of this pipeline does not apply

## Inputs

- Real PacBio HiFi amplicon reads for the Zymo/Titan-1 dataset (downloaded data file, FASTQ)
- MHASS-simulated reads matching the Zymo/Titan-1 amplicon composition (combined_reads.fastq from the simulation output directory)
- Reference amplicon sequences for alignment (e.g., amplicons.fa style FASTA)
- 16S-only negative-control datasets: ATCC 16S Data Set (Sequel II System 2.0 Release) and the Phylotag dataset

## Outputs

- Per-position insertion, substitution, and deletion error proportion table (one row per alignment position, per real/simulated read set)
- Overlaid real vs. simulated per-position error-rate traces per dataset, saved as insertion_rate_by_position.png

## How to apply

Align both the real amplicon reads and the corresponding MHASS-simulated reads (produced via MHASS's PBSIM3-based error model with a specified substitution:insertion:deletion difference-ratio) to their reference amplicon sequences. Using Python, tabulate insertion, substitution, and deletion counts at each alignment position and convert them to per-position error proportions for the real and simulated read sets. Repeat the same computation for 16S-only datasets (ATCC Sequel II 2.0 and Phylotag) to serve as negative controls where no position-specific insertion pattern is expected. Plot error rates versus alignment position in matplotlib with real and simulated traces overlaid per dataset. The decision point is whether the insertion proportion rises from baseline to >90% at approximately position ~2100 and stays elevated through the alignment end in both real and simulated Zymo/Titan-1 data while remaining at baseline in the 16S-only controls; save the resulting figure as insertion_rate_by_position.png.

## Related tools

- **MHASS** (Generates the simulated PacBio HiFi amplicon reads whose per-position error profile is compared against real reads (error model controlled by the PBSIM difference-ratio substitution:insertion:deletion parameter)) — https://github.com/rhowardstone/MHASS
- **Python** (Tabulates per-alignment-position insertion/substitution/deletion counts and plots the real vs. simulated error-rate traces with matplotlib)
- **bash** (Runs the MHASS_evaluation runner scripts (e.g., 1A_runner.sh) that produce the datasets fed into this analysis) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
bash scripts/1A_runner.sh
```

## Evaluation signals

- Insertion proportion in the Zymo/Titan-1 real reads rises from baseline to >90% at approximately alignment position ~2100 and remains elevated through the end of the alignment
- The MHASS-simulated Zymo/Titan-1 reads reproduce the same elevated insertion-rate profile, with the real vs. simulated traces overlapping in shape
- 16S-only negative controls (ATCC and Phylotag) show no such rise — insertion proportions stay at baseline across all alignment positions
- Per-position insertion + substitution + deletion proportions are internally consistent (each error class computed identically for real and simulated sets)
- The figure is saved under the expected filename insertion_rate_by_position.png

## Limitations

- The terminal-insertion profile is specific to the Zymo/Titan-1 amplicon; it is absent in 16S-only datasets, so a null result on other amplicons is expected rather than a failure
- The simulated profile depends on MHASS's PBSIM error-model settings, e.g., the --difference-ratio substitution:insertion:deletion parameter and --subread-accuracy, so different parameter choices change the insertion proportions
- MHASS's empirical number-of-passes distribution must be supplied (--np-distribution) when using the default empirical distribution type; missing it blocks simulation of the comparison reads

## Evidence

- [abstract] Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [methods] Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [readme] And the Phylotag dataset: https://genome.jgi.doe.gov/portal/PhyloTag/PhyloTag.home.html: "And the Phylotag dataset: https://genome.jgi.doe.gov/portal/PhyloTag/PhyloTag.home.html"
- [readme] Difference (error) ratio for PBSIM (substitution:insertion:deletion).: [evidence span withheld — anchor status: external_doc]
- [readme] Evaluation scripts and data for all analyses in our evaluation of MHASS: "Evaluation scripts and data for all analyses in our evaluation of MHASS"
