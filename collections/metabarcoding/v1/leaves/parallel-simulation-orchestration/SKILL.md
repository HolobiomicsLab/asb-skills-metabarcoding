---
name: parallel-simulation-orchestration
description: Use when simulating PacBio HiFi amplicon reads whose number-of-passes
  distribution must vary per read, but the underlying simulator (PBSIM3) accepts only
  a single np per invocation.
when_to_use_negative:
- Simulating Illumina or other non-PacBio short-read data — MHASS/PBSIM3 model PacBio
  HiFi circular-consensus subreads and their error profiles
- A single fixed pass number is acceptable for all reads — then PBSIM3 can be invoked
  once directly without the sampling/grouping orchestration
- No empirical or fitted num-passes distribution is available and the defaults (lognormal
  mu=3.88, sigma=1.22) are inappropriate for the instrument/run being simulated
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: MHASS
  role: Samples pass numbers from empirical or fitted lognormal distributions, groups
    reads by np, and orchestrates per-np PBSIM3 invocations in parallel
  repo: https://github.com/rhowardstone/MHASS
- name: PBSIM3
  role: Subread simulator invoked separately for each distinct sampled pass number,
    configured via --subread-accuracy and --difference-ratio
- name: Python
  role: Implements MHASS including the multiprocessing-based parallelization of per-np
    PBSIM3 runs (--threads, default 190/all CPUs)
- name: bash
  role: Runner-script environment (e.g. scripts/1A_runner.sh) in which the orchestrated
    simulation is executed
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:951ce8e7c47c9b439f54066087b0ba015b4bf0dc51daabcb8334166170e21374
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - PBSIM3
  - Python
  - bash
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5294117647058824
n_claims: 17
---

# Parallel Simulation Orchestration

## Summary

MHASS technique for driving PBSIM3, which supports only a single pass number per run, by sampling each simulated read's number of passes (np) from an empirical or fitted lognormal distribution and invoking PBSIM3 once per distinct np value, parallelized across CPU cores.

## When to use

Use when simulating PacBio HiFi amplicon reads whose number-of-passes distribution must vary per read, but the underlying simulator (PBSIM3) accepts only a single np per invocation.

## When NOT to use

- Simulating Illumina or other non-PacBio short-read data — MHASS/PBSIM3 model PacBio HiFi circular-consensus subreads and their error profiles
- A single fixed pass number is acceptable for all reads — then PBSIM3 can be invoked once directly without the sampling/grouping orchestration
- No empirical or fitted num-passes distribution is available and the defaults (lognormal mu=3.88, sigma=1.22) are inappropriate for the instrument/run being simulated

## Inputs

- Amplicon reference FASTA (amplicons.fa)
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- Empirical num-passes distribution TSV (for --np-distribution-type empirical) OR lognormal mu/sigma parameters with np-min/np-max bounds
- Optional barcode TSV (id, forward, reverse)
- Subread accuracy and difference (error) ratio settings for PBSIM3
- Real PacBio HiFi run data (e.g. ATCC 16S Data Set Sequel II System 2.0 Release) from which empirical distributions are extracted

## Outputs

- combined_reads.fastq with per-read pass numbers drawn from the sampled distribution
- counts.tsv (simulated ASV counts per sample)
- counts_meta.tsv (sample library sizes)
- sample_barcode_map.tsv (sample-to-barcode mapping)
- sequence_file_mapping.tsv (detailed template mappings)

## How to apply

First obtain a pass-number source: either a TSV empirical num-passes distribution extracted from real PacBio HiFi runs (e.g. the ATCC 16S Sequel II System 2.0 release, passed via --np-distribution), or a lognormal fit used when the empirical distribution is sparse or noisy (--np-distribution-type lognormal with defaults mu=3.88, sigma=1.22, clipped to np-min=2 and np-max=59). For each simulated read, draw an np value from the chosen distribution, replacing PBSIM3's default single-pass-per-read behavior. Group the sampled np values and invoke PBSIM3 separately for each distinct value, supplying --subread-accuracy (default 0.65) and an appropriate --difference-ratio (e.g. 22:45:33 for PacBio Sequel) so subreads carry the correct number of passes. Parallelize the per-np PBSIM3 invocations across cores (Python multiprocessing within the bash 1A_runner.sh environment) using --threads, defaulting to all CPUs. Finally, validate that the np distribution of the simulated reads reproduces the input empirical or fitted distribution.

## Related tools

- **MHASS** (Samples pass numbers from empirical or fitted lognormal distributions, groups reads by np, and orchestrates per-np PBSIM3 invocations in parallel) — https://github.com/rhowardstone/MHASS
- **PBSIM3** (Subread simulator invoked separately for each distinct sampled pass number, configured via --subread-accuracy and --difference-ratio)
- **Python** (Implements MHASS including the multiprocessing-based parallelization of per-np PBSIM3 runs (--threads, default 190/all CPUs))
- **bash** (Runner-script environment (e.g. scripts/1A_runner.sh) in which the orchestrated simulation is executed) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- The empirical pass-number distribution of the simulated reads reproduces the empirical or fitted lognormal input distribution used for sampling
- Every distinct sampled np value (within np-min=2 to np-max=59 for the lognormal default) has a corresponding grouped PBSIM3 invocation
- Read counts in combined_reads.fastq are consistent with counts.tsv and counts_meta.tsv library sizes per sample
- Wall-clock time scales with the --threads setting rather than running per-np PBSIM3 jobs serially
- Subread accuracy and error profile reflect the chosen --subread-accuracy (default 0.65) and --difference-ratio (e.g. 22:45:33 for PacBio Sequel)

## Limitations

- PBSIM3 supports only a single pass number per run, forcing one invocation per distinct np value and a grouping/parallelization layer in MHASS
- Empirical per-read pass-number distributions from real data can be sparse or noisy, requiring a fitted lognormal fallback instead
- Lognormal np sampling is bounded by --np-min (2) and --np-max (59); empirical mode requires a TSV distribution file and fails without one

## Evidence

- [abstract] by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] Type of np distribution: empirical (from file) or lognormal (default: empirical): [evidence span withheld — anchor status: external_doc]
- [readme] TSV file with empirical num-passes distribution (default: None): [evidence span withheld — anchor status: external_doc]
- [readme] Number of parallel threads to use for simulation (default: `190`): [evidence span withheld — anchor status: not_found]
- [methods] Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
