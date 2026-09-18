---
name: pbsim3-simulation-invocation
description: Use when generating PacBio HiFi amplicon subreads with realistic per-read
  pass-number distributions from real sequencing runs (e.g., ATCC 16S Sequel II System
  2.0 data), and PBSIM3's single-pass-per-run limitation must be bypassed.
when_to_use_negative:
- The target platform is not PacBio HiFi amplicon sequencing (e.g., ONT error profiles)
  — the difference-ratio defaults are PacBio-specific and MHASS targets HiFi amplicons
- No empirical or fitted np distribution is available and a single fixed pass number
  per PBSIM3 run suffices — the per-pass grouping machinery is unnecessary
- The input is already simulated FASTQ subreads rather than amplicon templates to
  be simulated
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: MHASS
  role: 'Wraps PBSIM3: samples pass numbers per read, groups reads by np, and invokes
    PBSIM3 separately per pass number in parallel'
  repo: https://github.com/rhowardstone/MHASS
- name: PBSIM3
  role: Subread simulator invoked once per distinct sampled pass number with configured
    accuracy and difference ratio
- name: Python
  role: Implementation language for the sampling module and multiprocessing across
    CPU cores
  repo: https://github.com/rhowardstone/MHASS
- name: bash
  role: Runner scripts (e.g., scripts/1A_runner.sh, scripts/setup.sh) in which the
    simulation executes
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:e10a62ee325e53551e00110f91d628b28c1b4c0ec21fa2b4fb598d3d9e423ae6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - PBSIM3
  - Python
  - bash
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# pbsim3-simulation-invocation

## Summary

Per-pass PBSIM3 subread simulation driven by sampled numbers of passes (np): because PBSIM3 supports only a single pass number per run, MHASS groups simulated reads by pass number drawn from empirical or lognormal distributions and invokes PBSIM3 separately for each distinct np, parallelized across CPU threads.

## When to use

Use when generating PacBio HiFi amplicon subreads with realistic per-read pass-number distributions from real sequencing runs (e.g., ATCC 16S Sequel II System 2.0 data), and PBSIM3's single-pass-per-run limitation must be bypassed.

## When NOT to use

- The target platform is not PacBio HiFi amplicon sequencing (e.g., ONT error profiles) — the difference-ratio defaults are PacBio-specific and MHASS targets HiFi amplicons
- No empirical or fitted np distribution is available and a single fixed pass number per PBSIM3 run suffices — the per-pass grouping machinery is unnecessary
- The input is already simulated FASTQ subreads rather than amplicon templates to be simulated

## Inputs

- Amplicon reference FASTA (amplicons.fa) with ASV sequences
- TSV empirical num-passes distribution (--np-distribution) when --np-distribution-type is empirical
- Lognormal np parameters (mu, sigma, np-min, np-max) when --np-distribution-type is lognormal
- Mean subread accuracy (default 0.65) and PBSIM difference ratio (e.g., 6:55:39 or 22:45:33)
- Real PacBio HiFi run data (ATCC 16S Data Set Sequel II System 2.0 Release) for extracting empirical np distributions

## Outputs

- Simulated PacBio HiFi subreads with per-read pass numbers matching the sampled distribution (combined_reads.fastq)
- counts.tsv: simulated ASV counts per sample
- sequence_file_mapping.tsv: detailed template mappings
- Verification that the simulated read pass-number distribution reproduces the empirical/fitted input distributions

## How to apply

First obtain the pass-number (np) source: either a TSV empirical np-distribution file supplied via --np-distribution (required when --np-distribution-type is empirical) or lognormal parameters via --lognormal-mu (default 3.88) and --lognormal-sigma (default 1.22), clamped to --np-min (2) and --np-max (59). For each simulated read, draw an np value from the chosen distribution, replacing PBSIM3's default single-pass-per-read behavior. Group reads by their sampled np value and invoke PBSIM3 once per distinct pass number, passing the subread accuracy (--subread-accuracy, default 0.65) and the error ratio (--difference-ratio; 6:55:39 PacBio RS II default, use 22:45:33 for PacBio Sequel HiFi-like chemistry). Parallelize the per-np PBSIM3 invocations across threads (--threads; default 190 or all CPUs) within the bash execution environment (e.g., 1A_runner.sh). Validate that the np distribution of the simulated reads reproduces the empirical or fitted lognormal input distribution.

## Related tools

- **MHASS** (Wraps PBSIM3: samples pass numbers per read, groups reads by np, and invokes PBSIM3 separately per pass number in parallel) — https://github.com/rhowardstone/MHASS
- **PBSIM3** (Subread simulator invoked once per distinct sampled pass number with configured accuracy and difference ratio)
- **Python** (Implementation language for the sampling module and multiprocessing across CPU cores) — https://github.com/rhowardstone/MHASS
- **bash** (Runner scripts (e.g., scripts/1A_runner.sh, scripts/setup.sh) in which the simulation executes) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
mhass --amplicon-fasta amplicons.fa --amplicon-genome-labels labels.tsv --barcode-file barcodes.tsv --np-distribution-type lognormal --lognormal-mu 3.88 --lognormal-sigma 1.22 --np-min 2 --np-max 59 --subread-accuracy 0.65 --threads 190 --output-dir testout
```

## Evaluation signals

- The np distribution of the simulated reads reproduces the empirical or fitted lognormal input distribution used for sampling
- All sampled np values fall within the configured bounds (--np-min default 2, --np-max default 59)
- Every simulated read was produced by a PBSIM3 run invoked at exactly its sampled pass number (no reads with default single-pass behavior)
- Simulated subreads respect the configured --subread-accuracy (default 0.65) and --difference-ratio error profile
- Output directory contains the expected artifacts: combined_reads.fastq, counts.tsv, counts_meta.tsv, sample_barcode_map.tsv, sequence_file_mapping.tsv

## Limitations

- PBSIM3 itself supports only a single pass number per run, so each distinct sampled np requires a separate PBSIM3 invocation, which multiplies process launches
- Empirical per-read pass-number distributions from real data can be sparse or noisy; in that case a fitted lognormal distribution is used as the sampling source instead
- The default difference ratio 6:55:39 is for PacBio RS II; using it for Sequel or ONT targets misrepresents the error profile unless overridden

## Evidence

- [readme] Type of np distribution: empirical (from file) or lognormal (default: empirical): [evidence span withheld — anchor status: external_doc]
- [readme] lognormal defaults: [evidence span withheld — anchor status: external_doc]
- [readme] parallelization: [evidence span withheld — anchor status: not_found]
- [abstract] empirically derived pass-number distributions: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] subread accuracy parameter: [evidence span withheld — anchor status: not_found]
