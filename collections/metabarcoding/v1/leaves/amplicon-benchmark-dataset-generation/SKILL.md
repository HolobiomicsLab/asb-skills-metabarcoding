---
name: amplicon-benchmark-dataset-generation
description: Use when building synthetic long-read (PacBio HiFi) amplicon read sets
  with known ground-truth ASV counts to benchmark ASV clustering, taxonomic assignment,
  or abundance-estimation workflows.
when_to_use_negative:
- The input is already an observed feature/count table — this skill simulates reads
  from genomes/amplicons, not from existing ASV tables.
- The target platform is short-read (e.g. Illumina) — MHASS generates PacBio HiFi
  amplicon data with PBSIM3 error profiles and pass-number distributions.
- No amplicon-to-genome mapping or genome abundance information is available, since
  the abundance model is genome-aware.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MHASS
  role: Simulator that performs genome-aware abundance modeling, dual barcoding, and
    pass-number simulation to emit synthetic HiFi FASTQ and ground-truth count tables
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language for MHASS with automated dependency management
- name: git
  role: Clones the MHASS_evaluation repository containing runner scripts and evaluation
    data
  repo: https://github.com/rhowardstone/MHASS_evaluation/
- name: bash
  role: Runs scripts/setup.sh for installation and scripts/1A_runner.sh to validate
    the simulation pipeline
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:5c5d99440458d4f197c735863a4d6f5990aa2795a1ec5db97aea5ea9da827bf4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# amplicon-benchmark-dataset-generation

## Summary

Generate realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome benchmarking by combining genome-aware abundance simulation (metaSPARSim-driven negative binomial between-sample variability) with dual-barcoding and empirically derived pass-number distributions.

## When to use

Use when building synthetic long-read (PacBio HiFi) amplicon read sets with known ground-truth ASV counts to benchmark ASV clustering, taxonomic assignment, or abundance-estimation workflows.

## When NOT to use

- The input is already an observed feature/count table — this skill simulates reads from genomes/amplicons, not from existing ASV tables.
- The target platform is short-read (e.g. Illumina) — MHASS generates PacBio HiFi amplicon data with PBSIM3 error profiles and pass-number distributions.
- No amplicon-to-genome mapping or genome abundance information is available, since the abundance model is genome-aware.

## Inputs

- Amplicon reference FASTA (e.g. amplicons.fa with >ASV001 headers)
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- User-supplied genome molarity/abundance values or --genome-distribution choice (uniform, lognormal, powerlaw, empirical:<file>)
- Optional barcode TSV (id, forward, reverse)
- Empirical number-of-passes distribution TSV (required if --np-distribution-type empirical)

## Outputs

- counts.tsv (simulated ASV counts per sample)
- counts_meta.tsv (sample library sizes)
- sample_barcode_map.tsv (mapping of samples to barcodes)
- sequence_file_mapping.tsv (detailed template mappings)
- combined_reads.fastq (final simulated HiFi read set)

## How to apply

Clone the MHASS_evaluation repository and run scripts/setup.sh, then supply an amplicon FASTA plus an amplicon-to-genome label TSV (asvid<TAB>genomeid). Fit a negative binomial model using metaSPARSim's R1 preset values (var-intercept 1.47565981333483, var-slope -0.909890963463704) to predict each ASV's squared coefficient of variation between samples from its expected abundance; when the modeled variability would be negative, emulate the NA dispersion estimate as zero so replicate abundances are identical before technical sampling is applied. Simulate genome-level abundances from user-supplied genome molarity values (--genome-distribution uniform, lognormal, powerlaw, or empirical:<file>) across --num-samples samples at --num-reads reads each, apply dual barcodes if a barcode TSV is given, and simulate number-of-passes from an empirical TSV or a lognormal fit (mu 3.88, sigma 1.22, np-min 2, np-max 59). Validate the module by updating download references in the runner scripts and executing bash scripts/1A_runner.sh.

## Related tools

- **MHASS** (Simulator that performs genome-aware abundance modeling, dual barcoding, and pass-number simulation to emit synthetic HiFi FASTQ and ground-truth count tables) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language for MHASS with automated dependency management)
- **git** (Clones the MHASS_evaluation repository containing runner scripts and evaluation data) — https://github.com/rhowardstone/MHASS_evaluation/
- **bash** (Runs scripts/setup.sh for installation and scripts/1A_runner.sh to validate the simulation pipeline)

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --barcode-file Test-data_ATCC_16S_MSA-1003/barcodes.tsv --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- counts.tsv contains exactly --num-samples rows and counts_meta.tsv library sizes match --num-reads (default 10000) per sample.
- The variability model uses metaSPARSim R1 preset defaults: var-intercept 1.47565981333483 and var-slope -0.909890963463704.
- Negative-modeled between-sample variability is emulated as zero, so replicate ASV abundances are identical before technical sampling is applied.
- Every ASV in counts.tsv maps to a genome via the amplicon-genome-labels TSV, confirming genome-aware (not ASV-marginal) abundance simulation.
- bash scripts/1A_runner.sh completes end-to-end after download references in the runner scripts are updated.

## Limitations

- metaSPARSim's NA dispersion-parameter estimates are converted to zero variability, meaning such ASVs show identical replicate abundances before technical sampling (no biological variability is modeled for them).
- The default subread accuracy (0.65) and difference ratio (6:55:39) target PacBio RS II; other platforms require adjusted --subread-accuracy and --difference-ratio (e.g. 22:45:33 for Sequel, 39:24:36 for ONT).
- An empirical number-of-passes TSV must be supplied when --np-distribution-type is empirical; the lognormal alternative is a fitted approximation (mu 3.88, sigma 1.22).

## Evidence

- [abstract] by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] Intercept for ASV variability model (controls baseline variation between samples) (default: 1.47565981333483): "--var-intercept VAR_INTERCEPT
                        Intercept for ASV variability model (controls baseline variation between samples)"
- [methods] git clone https://github.com/rhowardstone/MHASS_evaluation.git: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [full_text] when the sampled variability would be <0 it is emulated as NA, which metaSPARSim converts to 0, yielding no biological variability: [evidence span withheld — anchor status: not_found]
