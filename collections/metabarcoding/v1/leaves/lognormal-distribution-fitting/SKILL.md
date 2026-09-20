---
name: lognormal-distribution-fitting
description: Use when empirical per-read pass-number distributions extracted from
  real PacBio HiFi runs (e.g. ATCC 16S Sequel II System 2.0 data) are sparse or noisy,
  and you need a fitted lognormal np distribution for read simulation.
when_to_use_negative:
- The empirical pass-number distribution from the real run is dense and smooth enough
  to use directly via --np-distribution (empirical mode)
- The simulator target is not PacBio HiFi amplicon data and pass number is not a meaningful
  read attribute
- You need to simulate Illumina or short-read data where PBSIM3 subread pass modeling
  does not apply
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: MHASS
  role: Simulator that accepts --np-distribution-type lognormal with --lognormal-mu/--lognormal-sigma
    and samples per-read pass numbers before invoking PBSIM3 per distinct pass count
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language for distribution fitting, np sampling, and parallel
    per-pass PBSIM3 dispatch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:d95261a5d8295596dc6bcdd621e1daf7c669ff03b969a8050fcf908f97129f40
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# Lognormal distribution fitting

## Summary

Fits lognormal distributions to sparse or noisy empirical per-read pass-number distributions from PacBio HiFi runs, so that MHASS can sample a smooth number-of-passes (np) distribution to drive per-pass PBSIM3 subread simulation.

## When to use

Use when empirical per-read pass-number distributions extracted from real PacBio HiFi runs (e.g. ATCC 16S Sequel II System 2.0 data) are sparse or noisy, and you need a fitted lognormal np distribution for read simulation.

## When NOT to use

- The empirical pass-number distribution from the real run is dense and smooth enough to use directly via --np-distribution (empirical mode)
- The simulator target is not PacBio HiFi amplicon data and pass number is not a meaningful read attribute
- You need to simulate Illumina or short-read data where PBSIM3 subread pass modeling does not apply

## Inputs

- Empirical per-read pass-number (np) distribution extracted from real PacBio HiFi sequencing runs (e.g. ATCC 16S Sequel II System 2.0 data)
- TSV file of empirical num-passes distribution (when using --np-distribution-type empirical instead of fitting)
- Fitted lognormal parameters (mu, sigma) with optional np-min and np-max bounds

## Outputs

- Per-read sampled number-of-passes values drawn from the fitted lognormal distribution
- Per-pass-number PBSIM3 invocations producing simulated subreads with the correct number of passes
- Simulated PacBio HiFi reads whose pass-number distribution reproduces the fitted lognormal input

## How to apply

Start from the empirical pass-number distribution extracted from the real PacBio HiFi run (the ATCC 16S Data Set Sequel II System 2.0 Release). Where per-read pass counts are sparse or noisy, fit a lognormal distribution and pass its mu and sigma parameters to MHASS via --np-distribution-type lognormal, --lognormal-mu and --lognormal-sigma (defaults mu=3.88, sigma=1.22 from the empirical fit). Bound sampling with --np-min (default 2) and --np-max (default 59) to keep pass numbers in a realistic range. For each simulated read, draw an np value from this fitted distribution and group reads by distinct np so PBSIM3 can be invoked once per pass number in parallel across CPU cores. Alternatively, supply an empirical np TSV directly via --np-distribution when the empirical distribution is dense enough.

## Related tools

- **MHASS** (Simulator that accepts --np-distribution-type lognormal with --lognormal-mu/--lognormal-sigma and samples per-read pass numbers before invoking PBSIM3 per distinct pass count) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language for distribution fitting, np sampling, and parallel per-pass PBSIM3 dispatch)

## Examples

```
mhass --amplicon-fasta amplicons.fa --amplicon-genome-labels labels.txt --np-distribution-type lognormal --lognormal-mu 3.88 --lognormal-sigma 1.22 --np-min 2 --np-max 59 --output-dir testout
```

## Evaluation signals

- The pass-number distribution of the simulated reads reproduces the fitted lognormal distribution (shape and mass within np-min=2 to np-max=59)
- No sampled np falls outside the configured [np-min, np-max] bounds (default 2–59)
- Distinct sampled np values each receive their own PBSIM3 run, and reads carry the correct number of passes
- Per-pass PBSIM3 jobs execute in parallel across the configured --threads (default 190 or all CPUs)

## Limitations

- PBSIM3 itself supports only a single pass number per run, so the fitted distribution cannot be passed to PBSIM3 directly; reads must be grouped by np and simulated in separate invocations
- Sparse or noisy empirical distributions make direct empirical sampling unreliable, which is why fitting is needed in the first place
- Lognormal fitting imposes a parametric shape that may not capture multi-modal or truncated empirical np behavior; np-min/np-max clamping is a coarse correction

## Evidence

- [other] MHASS overcomes PBSIM3's single-pass-per-run restriction by sampling pass numbers from either empirical distributions extracted from real data or fitted lognormal distributions: [evidence span withheld — anchor status: not_found]
- [readme] --np-distribution-type flag: "--np-distribution-type {empirical,lognormal}
                        Type of np distribution: empirical (from file) or lognormal (default: empirical)"
- [readme] lognormal defaults from empirical fit: "--lognormal-mu LOGNORMAL_MU
                        mu parameter for lognormal distribution of Number of Passes (default from empirical fit) (default: 3.88)"
- [readme] np bounds: [evidence span withheld — anchor status: external_doc]
- [abstract] empirically derived pass-number distributions: "empirically derived pass-number distributions from actual sequencing runs"
