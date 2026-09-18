---
name: pacbio-hifi-pass-number-sampling
description: Use when simulating PacBio HiFi amplicon subreads whose number of passes
  (np) must match a real sequencing run's distribution, since PBSIM3 supports only
  one pass number per invocation.
when_to_use_negative:
- The simulation target is not PacBio HiFi amplicon data (e.g., ONT reads or short-read
  data), where PBSIM3 pass-number modeling is irrelevant.
- A single fixed pass number is acceptable for all simulated reads, in which case
  a plain PBSIM3 run suffices without distribution sampling.
- No empirical HiFi data or fitted lognormal parameters are available to define the
  number-of-passes distribution (an np source must exist).
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3307
tools:
- name: MHASS
  role: Implements pass-number sampling from empirical or lognormal distributions
    and orchestrates per-pass PBSIM3 runs
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language of MHASS; used for multiprocessing across pass-number
    groups
- name: PBSIM3
  role: Subread simulator invoked separately for each distinct sampled pass number
- name: bash
  role: Runner scripts (e.g., 1A_runner.sh) drive the sampling and simulation workflow
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:ef6b16b42e118ad2034482faa07be9be97ab843553d7bb246d5e6e3470d953da
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
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# pacbio-hifi-pass-number-sampling

## Summary

Samples per-read polymerase pass numbers from empirical distributions extracted from real PacBio HiFi runs (e.g., the ATCC 16S Sequel II System 2.0 dataset) or from fitted lognormal distributions, and invokes PBSIM3 separately for each distinct pass number in parallel so simulated subreads/CCS reads carry realistic np values despite PBSIM3's single-pass-per-run limitation.

## When to use

Use when simulating PacBio HiFi amplicon subreads whose number of passes (np) must match a real sequencing run's distribution, since PBSIM3 supports only one pass number per invocation.

## When NOT to use

- The simulation target is not PacBio HiFi amplicon data (e.g., ONT reads or short-read data), where PBSIM3 pass-number modeling is irrelevant.
- A single fixed pass number is acceptable for all simulated reads, in which case a plain PBSIM3 run suffices without distribution sampling.
- No empirical HiFi data or fitted lognormal parameters are available to define the number-of-passes distribution (an np source must exist).

## Inputs

- Amplicon reference FASTA (amplicons.fa)
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- Empirical num-passes distribution TSV (required if --np-distribution-type is empirical)
- Lognormal np parameters (mu=3.88, sigma=1.22, np-min=2, np-max=59) when using --np-distribution-type lognormal
- Real PacBio HiFi run data (e.g., ATCC 16S Sequel II System 2.0 release) to derive empirical pass-number distributions

## Outputs

- combined_reads.fastq with per-read pass numbers drawn from the sampled np distribution
- counts.tsv (simulated ASV counts per sample)
- counts_meta.tsv (sample library sizes)
- sample_barcode_map.tsv
- sequence_file_mapping.tsv (detailed template mappings)

## How to apply

First extract an empirical number-of-passes distribution from real HiFi data (e.g., the ATCC 16S Data Set Sequel II System 2.0 Release) into a TSV, or fit a lognormal (MHASS defaults mu=3.88, sigma=1.22, np-min=2, np-max=59) where the empirical distribution is sparse or noisy. Set --np-distribution-type to either 'empirical' (with --np-distribution pointing at the TSV) or 'lognormal' (with --lognormal-mu/--lognormal-sigma), and draw a pass number per simulated read. Group reads by sampled np value and run PBSIM3 once per distinct pass number — supplying the per-group np, --subread-accuracy (default 0.65) and a platform-appropriate --difference-ratio (6:55:39 for PacBio RS II, 22:45:33 for Sequel) — parallelizing the invocations across threads (--threads, defaults to all CPUs) within the 1A_runner.sh execution environment. Validate by comparing the pass-number distribution of the simulated reads against the input empirical/fitted distribution.

## Related tools

- **MHASS** (Implements pass-number sampling from empirical or lognormal distributions and orchestrates per-pass PBSIM3 runs) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language of MHASS; used for multiprocessing across pass-number groups)
- **PBSIM3** (Subread simulator invoked separately for each distinct sampled pass number)
- **bash** (Runner scripts (e.g., 1A_runner.sh) drive the sampling and simulation workflow) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- The pass-number distribution of the simulated reads reproduces the empirical/fitted input distribution used for sampling
- All sampled np values fall within the configured bounds (np-min=2, np-max=59 for the lognormal default)
- Reads generated by each per-np PBSIM3 invocation carry the correct number of passes for their group
- counts.tsv ASV counts per sample are consistent with --num-reads (default 10000 per sample)
- Subread accuracy centers on the value passed via --subread-accuracy (default 0.65)

## Limitations

- PBSIM3 supports only a single pass number per run, so reads must be grouped by sampled np and PBSIM3 invoked separately for each group
- Empirical per-read pass-number distributions from real runs can be sparse or noisy, requiring a lognormal fit as the sampling source
- PBSIM3 --difference-ratio defaults are PacBio RS II-specific (6:55:39); Sequel II HiFi simulations should use 22:45:33 for PacBio Sequel

## Evidence

- [abstract] by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] np distribution type options: [evidence span withheld — anchor status: external_doc]
- [readme] empirical np TSV requirement: "--np-distribution NP_DISTRIBUTION
                        TSV file with empirical num-passes distribution (default: None)"
- [methods] evaluation runner entry point: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
