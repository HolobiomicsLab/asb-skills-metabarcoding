---
name: metagenome-abundance-simulation
description: Use when generating realistic synthetic per-sample ASV abundances from
  user-supplied genome molarity values for PacBio HiFi microbiome simulations, especially
  when between-sample variability must follow metaSPARSim R1-preset dispersion.
when_to_use_negative:
- You already have a real ASV count feature table and only need sequencing-error or
  read-level simulation (e.g. PBSIM3 subread simulation with --subread-accuracy 0.65),
  not abundance synthesis
- You need variability fitted to your own cohort's dispersion rather than metaSPARSim's
  R1 preset values baked into MHASS defaults
- The input is a barcoding or num-passes (np) configuration problem rather than the
  abundance-variability step of the pipeline
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: MHASS
  role: Hosts the abundance-modeling code path; implements metaSPARSim-driven simulation
    of genome-level abundance matrices and exposes --var-intercept and --var-slope
    for the variability model
  repo: https://github.com/rhowardstone/MHASS
- name: metaSPARSim
  role: Provides the negative binomial R1 preset used to predict each ASV's squared
    coefficient of variation between samples from expected abundance; converts NA
    dispersion estimates to 0
- name: Python
  role: Language in which the abundance-modeling module of MHASS is implemented
- name: git
  role: Used to clone the MHASS_evaluation repository containing validation scripts
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: bash
  role: Runs scripts/setup.sh for setup and scripts/1A_runner.sh to validate the abundance
    simulation module
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:5c0624d9417cbc47c6f1137ba00b08a5459c8d09e3c49cdcaddcc20b97923c6b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# metagenome-abundance-simulation

## Summary

A genome-aware abundance simulation approach from MHASS that derives each ASV's between-sample variability from its expected abundance via a negative binomial model fit to metaSPARSim's R1 preset, producing synthetic abundance matrices for PacBio HiFi amplicon sequencing datasets.

## When to use

Use when generating realistic synthetic per-sample ASV abundances from user-supplied genome molarity values for PacBio HiFi microbiome simulations, especially when between-sample variability must follow metaSPARSim R1-preset dispersion.

## When NOT to use

- You already have a real ASV count feature table and only need sequencing-error or read-level simulation (e.g. PBSIM3 subread simulation with --subread-accuracy 0.65), not abundance synthesis
- You need variability fitted to your own cohort's dispersion rather than metaSPARSim's R1 preset values baked into MHASS defaults
- The input is a barcoding or num-passes (np) configuration problem rather than the abundance-variability step of the pipeline

## Inputs

- Amplicon FASTA file (amplicons.fa) with ASV reference sequences
- TSV file mapping amplicons to genomes (asvid<TAB>genomeid)
- User-supplied genome molarity values or a genome abundance distribution specification (uniform, lognormal, powerlaw, or empirical:<file.tsv>)
- Optional barcode TSV (id, forward, reverse)

## Outputs

- Simulated ASV abundance matrix per sample (counts.tsv)
- Sample library sizes (counts_meta.tsv)
- Genome-level abundance matrices from metaSPARSim simulation before technical sampling

## How to apply

Supply a FASTA of amplicon sequences plus a TSV mapping each ASV to a genome ID (asvid<TAB>genomeid), and choose a genome abundance distribution (uniform, lognormal, powerlaw, or empirical:<file>) from which genome molarity values are drawn. In the MHASS Python package, implement metaSPARSim-driven simulation of genome-level abundance matrices from these molarity values: fit a negative binomial model using metaSPARSim's R1 preset values (var-intercept 1.47565981333483, var-slope -0.909890963463704) to predict each ASV's squared coefficient of variation between samples from its expected abundance. When the sampled variability would be <0, emulate it as NA, which metaSPARSim converts to 0, yielding no biological variability so replicates are identical before technical sampling is applied. Control sample count and library size with --num-samples and --num-reads, then validate the module by executing bash scripts/1A_runner.sh from the MHASS_evaluation repository after updating download references in the runner scripts.

## Related tools

- **MHASS** (Hosts the abundance-modeling code path; implements metaSPARSim-driven simulation of genome-level abundance matrices and exposes --var-intercept and --var-slope for the variability model) — https://github.com/rhowardstone/MHASS
- **metaSPARSim** (Provides the negative binomial R1 preset used to predict each ASV's squared coefficient of variation between samples from expected abundance; converts NA dispersion estimates to 0)
- **Python** (Language in which the abundance-modeling module of MHASS is implemented)
- **git** (Used to clone the MHASS_evaluation repository containing validation scripts) — https://github.com/rhowardstone/MHASS_evaluation
- **bash** (Runs scripts/setup.sh for setup and scripts/1A_runner.sh to validate the abundance simulation module)

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --barcode-file Test-data_ATCC_16S_MSA-1003/barcodes.tsv --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- counts.tsv contains the expected number of samples (--num-samples) rows of ASV counts, and counts_meta.tsv library sizes are consistent with --num-reads
- Any ASV whose predicted squared coefficient of variation falls below 0 has zero variability, i.e. its replicate abundances are identical before technical sampling is applied (NA-to-zero emulation)
- The variability model uses R1-preset values: var-intercept 1.47565981333483 and var-slope -0.909890963463704 as the defaults for predicting squared CV from expected abundance
- Genome abundances are drawn from the selected --genome-distribution (uniform/lognormal/powerlaw/empirical:<file>) and respect user-supplied genome molarity values
- bash scripts/1A_runner.sh from MHASS_evaluation completes without error after repository setup and download-reference updates

## Limitations

- Negative dispersion estimates (sampled variability <0) are emulated as NA and converted to 0 by metaSPARSim, so those ASVs show no biological variability and identical replicates before technical sampling
- The variability model is anchored to metaSPARSim's R1 preset; other presets or data-derived dispersion would require changing --var-intercept/--var-slope
- The variability is a deterministic function of expected abundance (intercept + slope), so between-sample variability is abundance-dependent and not freely configurable per ASV

## Evidence

- [abstract] Genome-aware abundance modeling: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [methods] The abundance-modeling module fits a negative binomial model: [evidence span withheld — anchor status: not_found]
- [methods] metaSPARSim converts NA dispersion to zero variability: [evidence span withheld — anchor status: not_found]
- [readme] Variability model defaults in the CLI: [evidence span withheld — anchor status: external_doc]
- [readme] Genome abundance distribution options: [evidence span withheld — anchor status: external_doc]
