---
name: empirical-pass-number-distribution-extraction
description: Use when simulating PacBio HiFi amplicon data whose subread pass counts
  should reflect a real instrument run, or when real run data has sparse/noisy per-read
  pass counts and a lognormal fit (mu, sigma, np-min, np-max) is needed as the sampling
  source.
when_to_use_negative:
- The pass-number distribution of the target run is already faithfully represented
  by PBSIM3's default single-pass-per-run behavior and per-pass variation is irrelevant
  to the downstream benchmark
- The input is already a simulated dataset or a feature/ASV table with no underlying
  per-read pass counts to tabulate
edam_operation: http://edamontology.org/operation_0455
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: MHASS
  role: Consumes the empirical or lognormal pass-number distribution (--np-distribution-type,
    --np-distribution, --lognormal-mu, --lognormal-sigma, --np-min, --np-max), samples
    per-read pass numbers, and drives per-pass PBSIM3 invocations in parallel across
    threads
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language for the distribution extraction and per-pass-number
    sampling logic
  repo: https://github.com/rhowardstone/MHASS
- name: PacificBiosciences/DevNet
  role: Source of the real ATCC 16S Sequel II System 2.0 run data from which the empirical
    pass-number distributions are extracted
  repo: https://github.com/PacificBiosciences/DevNet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:9eec53758601af3fcd8033661582d37a6dd25e9f1f4ba3dcffc94cd52b049ca9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - PacificBiosciences/DevNet
  - Python
  edam_operation: http://edamontology.org/operation_0455
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Empirical pass-number distribution extraction

## Summary

Extract per-read number-of-passes (np) distributions from real PacBio HiFi sequencing runs (e.g., the ATCC 16S Sequel II System 2.0 release) and supply them, or a fitted lognormal surrogate, to MHASS so PBSIM3 can generate reads with realistic pass counts.

## When to use

Use when simulating PacBio HiFi amplicon data whose subread pass counts should reflect a real instrument run, or when real run data has sparse/noisy per-read pass counts and a lognormal fit (mu, sigma, np-min, np-max) is needed as the sampling source.

## When NOT to use

- The pass-number distribution of the target run is already faithfully represented by PBSIM3's default single-pass-per-run behavior and per-pass variation is irrelevant to the downstream benchmark
- The input is already a simulated dataset or a feature/ASV table with no underlying per-read pass counts to tabulate

## Inputs

- Real PacBio HiFi sequencing run (e.g., ATCC 16S Data Set Sequel II System 2.0 Release BAM/subreads with per-read pass counts)
- Empirical num-passes distribution as a TSV file (required when --np-distribution-type is empirical)
- Lognormal fit parameters (mu, sigma, np-min, np-max) when the empirical distribution is sparse or noisy

## Outputs

- Empirical number-of-passes distribution TSV consumable by MHASS via --np-distribution
- Fitted lognormal parameters (mu, sigma) with clipping bounds np-min and np-max
- Simulated HiFi reads whose pass-number distribution reproduces the empirical/fitted input distribution

## How to apply

Load the real PacBio HiFi run (e.g., the ATCC 16S Data Set Sequel II System 2.0 Release from PacificBiosciences/DevNet) and tabulate the per-read number of passes into an empirical distribution TSV. Where the empirical distribution is sparse or noisy, fit a lognormal distribution to it and use the fitted parameters instead (MHASS defaults derived from empirical fit: mu=3.88, sigma=1.22, np-min=2, np-max=59). Pass the result to MHASS via --np-distribution-type (empirical or lognormal), --np-distribution (the empirical TSV), or --lognormal-mu/--lognormal-sigma/--np-min/--np-max. For simulation, per-read pass numbers are drawn from this distribution, and PBSIM3 is invoked separately for each distinct pass number so reads carry the correct number of passes. Validate that the pass-number distribution of simulated reads reproduces the input empirical/fitted distribution.

## Related tools

- **MHASS** (Consumes the empirical or lognormal pass-number distribution (--np-distribution-type, --np-distribution, --lognormal-mu, --lognormal-sigma, --np-min, --np-max), samples per-read pass numbers, and drives per-pass PBSIM3 invocations in parallel across threads) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language for the distribution extraction and per-pass-number sampling logic) — https://github.com/rhowardstone/MHASS
- **PacificBiosciences/DevNet** (Source of the real ATCC 16S Sequel II System 2.0 run data from which the empirical pass-number distributions are extracted) — https://github.com/PacificBiosciences/DevNet

## Examples

```
mhass --amplicon-fasta amplicons.fa --amplicon-genome-labels labels.tsv --output-dir output/ --np-distribution-type empirical --np-distribution np_dist.tsv --threads 16
```

## Evaluation signals

- The empirical num-passes TSV is well-formed and accepted by MHASS (--np-distribution-type empirical runs without error)
- Simulated reads' pass-number distribution statistically matches the input empirical or fitted lognormal distribution
- With --np-distribution-type lognormal, sampled np values fall within the configured np-min/np-max bounds (defaults 2 to 59)
- Distinct sampled pass numbers each map to a separate PBSIM3 run, so reads are generated with the correct number of passes
- Alternative error-ratio and accuracy settings (e.g., --subread-accuracy 0.65) remain consistent across per-pass PBSIM3 invocations

## Limitations

- Empirical per-read pass-number distributions from real runs can be sparse or noisy, requiring a lognormal fit as a surrogate
- PBSIM3 itself supports only a single pass number per run, so per-pass grouping and parallel invocation across CPU cores is required
- Lognormal sampling is clipped to np-min/np-max (defaults 2 and 59), which truncates distribution tails

## Evidence

- [abstract] empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [methods] ATCC dataset source: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [readme] distribution type options: [evidence span withheld — anchor status: not_found]
- [readme] empirical TSV requirement: [evidence span withheld — anchor status: not_found]
