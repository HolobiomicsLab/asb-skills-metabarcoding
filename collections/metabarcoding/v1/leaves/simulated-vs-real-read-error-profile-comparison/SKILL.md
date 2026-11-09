---
name: simulated-vs-real-read-error-profile-comparison
description: 'Use when evaluating whether a HiFi amplicon simulator reproduces real-read
  error behavior: you have raw real and simulated FASTQ reads plus reference sequences,
  and need per-position insertion/substitution/deletion profiles to detect position-specific
  error signatures.'
when_to_use_negative:
- Input is already an ASV/feature count table (counts.tsv, counts_meta.tsv) rather
  than raw reads — no alignment to a reference is possible, so per-position error
  decomposition cannot be computed
- Reads cannot be aligned to a reference sequence, since the skill requires position-resolved
  error counts along the alignment
- The goal is a global (non position-specific) error rate summary rather than a position-by-position
  real-vs-simulated comparison
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MHASS
  role: Generates the synthetic PacBio HiFi amplicon reads whose error profile is
    compared against real Zymo/Titan-1 reads
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Tabulates insertion/substitution/deletion counts per alignment position and
    plots per-position error rates with matplotlib
- name: bash
  role: Runs the MHASS_evaluation setup and runner scripts that produce/simulate the
    read sets and downloads used in this analysis
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:42da4b339b7c51b04c22f8fa3860bc4bf43ee267f5506f7f8ba452e7175c3de2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# simulated-vs-real-read-error-profile-comparison

## Summary

Compares per-position error profiles (insertion/substitution/deletion proportions) between real PacBio HiFi amplicon reads and MHASS-simulated reads by aligning both read sets to their references and tabulating error types at each alignment position, to verify the simulator reproduces position-specific error signatures such as the Zymo/Titan-1 terminal-insertion profile.

## When to use

Use when evaluating whether a HiFi amplicon simulator reproduces real-read error behavior: you have raw real and simulated FASTQ reads plus reference sequences, and need per-position insertion/substitution/deletion profiles to detect position-specific error signatures.

## When NOT to use

- Input is already an ASV/feature count table (counts.tsv, counts_meta.tsv) rather than raw reads — no alignment to a reference is possible, so per-position error decomposition cannot be computed
- Reads cannot be aligned to a reference sequence, since the skill requires position-resolved error counts along the alignment
- The goal is a global (non position-specific) error rate summary rather than a position-by-position real-vs-simulated comparison

## Inputs

- Real Zymo/Titan-1 amplicon HiFi reads (FASTQ from the downloaded MHASS_evaluation data file)
- MHASS-simulated reads for the same amplicon (combined_reads.fastq from the simulation output directory)
- Reference amplicon sequences for read alignment
- 16S-only ATCC dataset reads (PacBio DevNet Sequel II System 2.0 release) as negative control
- Phylotag dataset reads (JGI PhyloTag) as negative control

## Outputs

- Per-position error proportion table (insertion, substitution, deletion counts and proportions at each alignment position) for real and simulated read sets
- Figure insertion_rate_by_position.png with real vs simulated error-rate traces overlaid per dataset

## How to apply

Load the real Zymo/Titan-1 amplicon reads (from the downloaded MHASS_evaluation data file) and the corresponding MHASS-simulated reads, and align both sets to their reference sequences. Using Python, tabulate insertion, substitution, and deletion counts at each alignment position and convert them to per-position error proportions for both read sets. Repeat the same per-position error-rate computation for the 16S-only ATCC (PacBio DevNet Sequel II 2.0 release) and Phylotag datasets as negative controls. Plot error rates versus alignment position with matplotlib, overlaying real vs simulated traces per dataset. The decision point for validation is whether the insertion proportion rises from baseline to >90% at approximately position ~2100 through the alignment end in both Zymo/Titan-1 real and simulated reads, and whether this rise is absent in 16S-only datasets.

## Related tools

- **MHASS** (Generates the synthetic PacBio HiFi amplicon reads whose error profile is compared against real Zymo/Titan-1 reads) — https://github.com/rhowardstone/MHASS
- **Python** (Tabulates insertion/substitution/deletion counts per alignment position and plots per-position error rates with matplotlib)
- **bash** (Runs the MHASS_evaluation setup and runner scripts that produce/simulate the read sets and downloads used in this analysis) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
bash scripts/1A_runner.sh
```

## Evaluation signals

- The proportion of insertions rises from baseline to >90% at approximately position ~2100 and remains elevated through the end of the alignment for Zymo/Titan-1 real reads
- The same terminal insertion rise appears in the MHASS-simulated Zymo/Titan-1 reads, confirming simulator fidelity
- The 16S-only ATCC and Phylotag negative-control datasets show no such insertion rise across the alignment
- Real vs simulated error-rate traces for each dataset are overlaid in the saved figure insertion_rate_by_position.png
- Error proportions at every position sum across insertion/substitution/deletion categories consistently for both real and simulated read sets

## Limitations

- The >90% terminal insertion signature at position ~2100 is specific to the Zymo/Titan-1 amplicon; it is absent in 16S-only datasets, so success of the comparison cannot be judged by that threshold on 16S controls
- Analysis depends on aligning reads to correct reference sequences; reference or alignment errors would corrupt the per-position error proportions
- The negative-control datasets must be downloaded separately (ATCC from PacBio DevNet, Phylotag from JGI), and runner-script references must be updated to the local download locations

## Evidence

- [results] Both real and simulated Zymo/Titan-1 data show elevated insertion rates, with the proportion of insertions rising from baseline to >90% at position ∼2100 through the end of the alignment; this pattern is absent in 16S-only datasets.: [evidence span withheld — anchor status: not_found]
- [methods] MHASS simulated reads and evaluation data provenance: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [readme] MHASS output FASTQ and error model parameters: [evidence span withheld — anchor status: external_doc]
- [readme] Dataset download instructions for negative controls: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [readme] Simulated reads produced as FASTQ output: [evidence span withheld — anchor status: external_doc]
