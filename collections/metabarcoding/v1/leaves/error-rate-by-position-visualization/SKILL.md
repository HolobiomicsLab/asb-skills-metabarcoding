---
name: error-rate-by-position-visualization
description: Use when aligned HiFi amplicon reads (real or simulated, e.g. MHASS output)
  must be checked for position-specific error patterns such as terminal insertion
  spikes that are invisible in aggregate error rates.
when_to_use_negative:
- Input reads are unaligned FASTQ with no reference alignment available — per-position
  error tabulation requires alignments to reference sequences
- The goal is a single aggregate substitution:insertion:deletion ratio (e.g. to set
  MHASS --difference-ratio) rather than a position-resolved error profile
- Amplicons shorter than the ~2100 bp region of interest, where the terminal-insertion
  profile cannot manifest
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: MHASS
  role: Generates the simulated PacBio HiFi amplicon reads whose per-position error
    profile is compared against real reads
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Tabulates insertion/substitution/deletion counts per alignment position and
    converts them to error proportions; implemented with matplotlib
- name: bash
  role: Runs the evaluation runner scripts (e.g. 1A_runner.sh) that produce the aligned
    read sets for analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:e24454bc242bb380ffb61d13f6b2659651fc33fb5a918d61fb1b9d06ed054314
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# error-rate-by-position-visualization

## Summary

Computes per-alignment-position insertion, substitution, and deletion error proportions from aligned PacBio HiFi amplicon reads and plots error rate versus alignment position (matplotlib) to expose position-specific error profiles, e.g. the terminal-insertion spike in the Zymo/Titan-1 amplicon where insertions rise to >90% near position ~2100.

## When to use

Use when aligned HiFi amplicon reads (real or simulated, e.g. MHASS output) must be checked for position-specific error patterns such as terminal insertion spikes that are invisible in aggregate error rates.

## When NOT to use

- Input reads are unaligned FASTQ with no reference alignment available — per-position error tabulation requires alignments to reference sequences
- The goal is a single aggregate substitution:insertion:deletion ratio (e.g. to set MHASS --difference-ratio) rather than a position-resolved error profile
- Amplicons shorter than the ~2100 bp region of interest, where the terminal-insertion profile cannot manifest

## Inputs

- Aligned reads (SAM/BAM) for real Zymo/Titan-1 amplicon reads (downloaded data file)
- Aligned MHASS-simulated reads for the same Zymo/Titan-1 amplicon
- Aligned 16S-only ATCC dataset reads (Sequel II System 2.0 Release)
- Aligned Phylotag dataset reads
- Reference sequences for each amplicon/dataset

## Outputs

- Per-position error proportion tables (insertion, substitution, deletion) for real and simulated read sets
- insertion_rate_by_position.png — matplotlib figure with real vs simulated error-rate traces overlaid per dataset
- Validation verdict: presence/absence of the terminal insertion spike (>90% insertion at ~position 2100)

## How to apply

Align the real reads (e.g. the Zymo/Titan-1 amplicon, 16S-only ATCC, and Phylotag datasets) and the corresponding MHASS-simulated reads to their reference sequences, then use Python to tabulate insertion, substitution, and deletion counts at each alignment position and convert them to per-position error proportions. Repeat the computation for 16S-only datasets as negative controls. Plot error rates versus alignment position in matplotlib, overlaying real vs simulated traces per dataset, and save the figure as insertion_rate_by_position.png. Validate that for Zymo/Titan-1 both real and simulated traces show the insertion proportion rising from baseline to >90% at approximately position 2100 and remaining elevated through the alignment end, while 16S-only datasets stay at baseline throughout.

## Related tools

- **MHASS** (Generates the simulated PacBio HiFi amplicon reads whose per-position error profile is compared against real reads) — https://github.com/rhowardstone/MHASS
- **Python** (Tabulates insertion/substitution/deletion counts per alignment position and converts them to error proportions; implemented with matplotlib)
- **bash** (Runs the evaluation runner scripts (e.g. 1A_runner.sh) that produce the aligned read sets for analysis)

## Examples

```
import matplotlib.pyplot as plt; plt.plot(pos, ins_real, label='real'); plt.plot(pos, ins_sim, label='simulated'); plt.xlabel('alignment position'); plt.savefig('insertion_rate_by_position.png')
```

## Evaluation signals

- Insertion proportion rises from baseline to >90% at approximately position ~2100 in BOTH real and simulated Zymo/Titan-1 traces and stays elevated through the alignment end
- No corresponding rise in insertion rate appears in the 16S-only ATCC or Phylotag negative-control traces
- Real and simulated error-rate curves overlap closely across positions for each dataset
- Per-position proportions for insertion, substitution, and deletion each sum appropriately across positions and are defined for every alignment position through the end of the alignment
- Figure is saved as insertion_rate_by_position.png with real vs simulated traces clearly overlaid per dataset

## Limitations

- The terminal-insertion profile is amplicon-specific: it appears in Zymo/Titan-1 but is absent in 16S-only datasets, so failure to reproduce it in other amplicons may be expected rather than a bug
- Simulated profile fidelity depends on MHASS PBSIM3 parameters (e.g. --difference-ratio, --subread-accuracy) matching the instrument and library of the real data
- Requires alignments to reference sequences; per-position error rates cannot be computed directly from unaligned FASTQ

## Evidence

- [methods] Using Python, tabulate insertion, substitution, and deletion counts at each alignment position and convert to per-position error proportions for the real and simulated Zymo/Titan-1 read sets.: [evidence span withheld — anchor status: not_found]
- [methods] Plot error rates versus alignment position with real vs simulated traces overlaid and save the figure.: [evidence span withheld — anchor status: not_found]
- [methods] Validation criterion for the terminal-insertion profile in Zymo/Titan-1.: [evidence span withheld — anchor status: not_found]
- [results] The negative-control expectation in 16S-only datasets.: [evidence span withheld — anchor status: not_found]
- [readme] MHASS output format consumed by the analysis pipeline.: [evidence span withheld — anchor status: external_doc]
