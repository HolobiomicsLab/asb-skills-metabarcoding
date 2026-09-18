---
name: negative-binomial-count-modeling
description: Use when generating synthetic microbiome count/abundance matrices that
  need realistic between-sample biological variability per ASV, e.g. simulating PacBio
  HiFi amplicon data from genome molarity values.
when_to_use_negative:
- You already have an observed ASV count table and want downstream analysis rather
  than simulation of abundances.
- You need to simulate read-level technical error (PBSIM3 subread accuracy, difference-ratio
  substitution:insertion:deletion) — that is a separate MHASS step, not the abundance-variability
  model.
- The target ASVs have abundance levels where the fitted squared CV would be negative
  and you require nonzero variability — the NA-to-zero emulation intentionally yields
  zero biological variability for those ASVs.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MHASS
  role: Host pipeline whose abundance-modeling module implements the metaSPARSim-driven
    negative binomial variability simulation
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language for the genome-aware abundance simulation code path
    in the MHASS package
- name: metaSPARSim
  role: Source of the R1 preset values for the negative binomial between-sample variability
    model; its NA-to-zero dispersion conversion is emulated
- name: git
  role: Used to clone the MHASS_evaluation repository to obtain evaluation scripts
    and data
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: bash
  role: Runs scripts/setup.sh and scripts/1A_runner.sh to install dependencies and
    validate the simulation module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:4be715d7be98171b12ed10078199aa7ae3ec51da7be881c63e5e8dd214146607
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  - git
  - metaSPARSim
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# negative-binomial-count-modeling

## Summary

MHASS's genome-aware abundance simulation fits a negative binomial model using metaSPARSim's R1 preset to predict each ASV's between-sample squared coefficient of variation from its expected abundance, with NA dispersion estimates emulated as zero variability.

## When to use

Use when generating synthetic microbiome count/abundance matrices that need realistic between-sample biological variability per ASV, e.g. simulating PacBio HiFi amplicon data from genome molarity values.

## When NOT to use

- You already have an observed ASV count table and want downstream analysis rather than simulation of abundances.
- You need to simulate read-level technical error (PBSIM3 subread accuracy, difference-ratio substitution:insertion:deletion) — that is a separate MHASS step, not the abundance-variability model.
- The target ASVs have abundance levels where the fitted squared CV would be negative and you require nonzero variability — the NA-to-zero emulation intentionally yields zero biological variability for those ASVs.

## Inputs

- User-supplied genome molarity values (genome-level abundances)
- Amplicon FASTA file of ASV reference sequences (amplicons.fa)
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- metaSPARSim R1 preset values (exposed as MHASS --var-intercept 1.47565981333483 and --var-slope -0.909890963463704)

## Outputs

- Simulated ASV abundance/count matrix with genome-aware between-sample variability (counts.tsv in the MHASS output directory)
- Sample library sizes (counts_meta.tsv)

## How to apply

Implement the abundance-modeling step inside the MHASS Python package so that genome-level abundance matrices are simulated from user-supplied genome molarity values via metaSPARSim-driven simulation. Fit a negative binomial model to metaSPARSim's R1 preset — MHASS exposes this as the variability model with default intercept 1.47565981333483 (CLI flag --var-intercept) and slope -0.909890963463704 (CLI flag --var-slope), predicting each ASV's squared coefficient of variation between samples from its expected abundance. Sample a variability value per ASV; when the sampled value would be < 0, emulate metaSPARSim's behavior by treating it as NA, which metaSPARSim converts to 0 — i.e. no biological variability, so replicates of that ASV are identical before technical sampling is applied. After implementing, update the download references in the runner scripts and validate the module by executing bash scripts/1A_runner.sh.

## Related tools

- **MHASS** (Host pipeline whose abundance-modeling module implements the metaSPARSim-driven negative binomial variability simulation) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language for the genome-aware abundance simulation code path in the MHASS package)
- **metaSPARSim** (Source of the R1 preset values for the negative binomial between-sample variability model; its NA-to-zero dispersion conversion is emulated)
- **git** (Used to clone the MHASS_evaluation repository to obtain evaluation scripts and data) — https://github.com/rhowardstone/MHASS_evaluation
- **bash** (Runs scripts/setup.sh and scripts/1A_runner.sh to install dependencies and validate the simulation module)

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- ASVs whose sampled variability would be < 0 show identical replicate abundances before technical sampling is applied (NA-to-zero emulation working).
- Predicted squared coefficient of variation decreases with ASV expected abundance, matching the negative default slope -0.909890963463704.
- The variability model defaults match metaSPARSim's R1 preset (var-intercept 1.47565981333483, var-slope -0.909890963463704).
- bash scripts/1A_runner.sh completes without error after updating runner-script download references.
- Output directory contains counts.tsv, counts_meta.tsv, sample_barcode_map.tsv, sequence_file_mapping.tsv, and combined_reads.fastq.

## Limitations

- When the sampled variability would be < 0 it is emulated as NA and converted to 0, so such ASVs get no biological variability (identical replicates before technical sampling).
- The variability model is a two-parameter (intercept/slope) fit to metaSPARSim's R1 preset; it does not capture other dispersion regimes without overriding --var-intercept/--var-slope.
- PBSIM-related defaults are instrument-specific (difference-ratio 6:55:39 for PacBio RS II; 22:45:33 for Sequel), which is distinct from but downstream of the abundance model.

## Evidence

- [full_text] The abundance-modeling module fits a negative binomial model, using metaSPARSim's R1 preset values, to predict each ASV's squared coefficient of variation between samples from its expected abundance: [evidence span withheld — anchor status: not_found]
- [full_text] when the sampled variability would be <0 it is emulated as NA, which metaSPARSim converts to 0, yielding no biological variability (identical replicate abundances before technical sampling): [evidence span withheld — anchor status: not_found]
- [readme] --var-intercept VAR_INTERCEPT Intercept for ASV variability model (controls baseline variation between samples) (default: 1.47565981333483): [evidence span withheld — anchor status: external_doc]
