---
name: 16s-amplicon-error-benchmarking
description: Use when aligning real vs simulated PacBio HiFi amplicon reads (e.g.
  Zymo/Titan-1, ATCC 16S, Phylotag) to their references and you need to test whether
  error profiles — especially terminal insertion spikes — are reproduced by a simulator.
when_to_use_negative:
- Input is already an ASV/feature abundance table — use the abundance-correlation
  benchmarking workflow instead
- Reads are short-read or non-PacBio HiFi; the PBSIM difference-ratio model and HiFi
  circular-consensus error profile would not apply
- No reference amplicon sequences are available, since per-position error tabulation
  requires aligning reads to references
edam_operation: http://edamontology.org/operation_2403
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MHASS
  role: Simulates PacBio HiFi amplicon reads (via PBSIM3) whose per-position error
    profile is compared against real sequencing data
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Tabulates per-position error counts from alignments and computes error proportions;
    MHASS itself is implemented in Python
  repo: https://github.com/rhowardstone/MHASS
- name: bash
  role: Runs setup.sh and 1A_runner.sh to build the environment and execute the simulator
    on the three datasets
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: git
  role: Clones the MHASS_evaluation repository containing runner scripts and evaluation
    data
  repo: https://github.com/rhowardstone/MHASS_evaluation
merged_aliases:
- 16s-amplicon-benchmark-evaluation
merged_alias_records:
- alias: 16s-amplicon-benchmark-evaluation
  slug: 16s-amplicon-benchmark-evaluation
  cosine_score: 0.8805
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:1fa2728c2dd3dde4e37141037b40cea70d7c05e734b98ed079e0d97cf2c8ecb2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_2403
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# 16s-amplicon-error-benchmarking

## Summary

Per-position error-profiling workflow that quantifies insertion/substitution/deletion rates across alignment position for PacBio HiFi 16S amplicon reads, comparing real sequencing data against MHASS/PBSIM-simulated reads to reproduce terminal-insertion error profiles.

## When to use

Use when aligning real vs simulated PacBio HiFi amplicon reads (e.g. Zymo/Titan-1, ATCC 16S, Phylotag) to their references and you need to test whether error profiles — especially terminal insertion spikes — are reproduced by a simulator.

## When NOT to use

- Input is already an ASV/feature abundance table — use the abundance-correlation benchmarking workflow instead
- Reads are short-read or non-PacBio HiFi; the PBSIM difference-ratio model and HiFi circular-consensus error profile would not apply
- No reference amplicon sequences are available, since per-position error tabulation requires aligning reads to references

## Inputs

- Real PacBio HiFi amplicon reads (e.g. Zymo/Titan-1 mock community FASTQ/BAM)
- MHASS-simulated reads (combined_reads.fastq from MHASS output directory)
- Reference amplicon sequences (amplicons.fa) for alignment
- 16S-only negative-control read sets (ATCC DevNet Sequel II 2.0, Phylotag)

## Outputs

- Per-position insertion/substitution/deletion proportion tables for real and simulated reads
- insertion_rate_by_position.png (matplotlib overlay of real vs simulated error traces per dataset)

## How to apply

Set up the MHASS_evaluation environment (git clone, bash scripts/setup.sh), download the Zymo/Titan-1 data file, ATCC 16S DevNet dataset, and Phylotag dataset, then run bash scripts/1A_runner.sh to generate MHASS-simulated reads. Align both real and simulated reads to their reference amplicon sequences, and with Python tabulate insertion, substitution, and deletion counts per alignment position, converting them to per-position error proportions. Use 16S-only datasets (ATCC, Phylotag) as negative controls since they lack the full-length alignment span of Zymo/Titan-1. Plot error proportion versus alignment position in matplotlib with real and simulated traces overlaid per dataset; the key decision point is whether the simulated insertion profile matches the real data's terminal spike.

## Related tools

- **MHASS** (Simulates PacBio HiFi amplicon reads (via PBSIM3) whose per-position error profile is compared against real sequencing data) — https://github.com/rhowardstone/MHASS
- **Python** (Tabulates per-position error counts from alignments and computes error proportions; MHASS itself is implemented in Python) — https://github.com/rhowardstone/MHASS
- **bash** (Runs setup.sh and 1A_runner.sh to build the environment and execute the simulator on the three datasets) — https://github.com/rhowardstone/MHASS_evaluation
- **git** (Clones the MHASS_evaluation repository containing runner scripts and evaluation data) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git && cd MHASS_evaluation && bash scripts/setup.sh && bash scripts/1A_runner.sh
```

## Evaluation signals

- In Zymo/Titan-1 data (real and simulated), the proportion of insertions rises from baseline to >90% at approximately position ~2100 and stays elevated through the end of the alignment
- The terminal insertion spike is absent in both 16S-only negative controls (ATCC and Phylotag datasets)
- Real and simulated traces overlay closely per dataset when error proportion is plotted against alignment position
- MHASS output directory contains the expected files (combined_reads.fastq, counts.tsv, sequence_file_mapping.tsv), confirming the simulation ran correctly before alignment
- The figure is saved as insertion_rate_by_position.png with all six real/simulated traces represented

## Limitations

- The error profile is amplicon- and platform-specific: MHASS's default PBSIM difference ratio (6:55:39) targets PacBio RS II and must be changed (22:45:33 for Sequel) to match HiFi data, so mismatched ratios distort the insertion profile
- Only datasets with full-length alignment span (like Zymo/Titan-1) show the terminal insertion effect; 16S-only controls cannot reproduce it
- MHASS requires conda and samtools for installation

## Evidence

- [abstract] Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [methods] clone evaluation repo: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [methods] runner script: "bash scripts/1A_runner.sh"
- [readme] difference ratio: [evidence span withheld — anchor status: external_doc]
- [readme] MHASS output files: [evidence span withheld — anchor status: external_doc]
- [readme] setup instructions: "git clone https://github.com/rhowardstone/MHASS_evaluation.git
cd MHASS_evaluation
bash scripts/setup.sh"
