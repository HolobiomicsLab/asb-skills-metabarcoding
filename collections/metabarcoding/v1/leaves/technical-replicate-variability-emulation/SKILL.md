---
name: technical-replicate-variability-emulation
description: Use when simulating between-sample abundance variability for genome-derived
  ASV counts in MHASS-style PacBio HiFi amplicon data, i.e. when converting genome
  molarity values into replicate-specific abundance matrices.
when_to_use_negative:
- The input is already an observed ASV count/feature table — variability should come
  from data, not from the metaSPARSim R1-preset model.
- You need true biological replicate heterogeneity rather than MHASS's emulated model
  where negative-dispersion samples become identical replicates before technical sampling.
- Simulating sequencing platform noise (read errors, number of passes) rather than
  between-sample abundance variability — use the PBSIM3 and num-passes (empirical
  or lognormal, mu 3.88 / sigma 1.22) components instead.
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_2269
tools:
- name: MHASS
  role: Provides the abundance-modeling module that fits the metaSPARSim R1-preset
    negative binomial variability model and applies NA-to-zero dispersion emulation
  repo: https://github.com/rhowardstone/MHASS
- name: metaSPARSim
  role: Source of the R1 preset negative binomial model used to predict ASV squared
    coefficient of variation from expected abundance; also converts NA dispersion
    estimates to 0
- name: MHASS_evaluation
  role: Evaluation repository whose workflow (1A_runner.sh) validates the implemented
    abundance simulation module
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:c221b27f2b826f162853a5f5b313f1e0bb7ca4c9176495552f8d0a71f30145f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - MHASS_evaluation
  - Python
  - git
  - metaSPARSim
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# technical-replicate-variability-emulation

## Summary

Implements MHASS's genome-aware abundance variability step: a negative binomial model using metaSPARSim's R1 preset values predicts each ASV's squared coefficient of variation between samples from its expected abundance, with NA dispersion estimates emulated as zero variability so replicates are identical before technical sampling.

## When to use

Use when simulating between-sample abundance variability for genome-derived ASV counts in MHASS-style PacBio HiFi amplicon data, i.e. when converting genome molarity values into replicate-specific abundance matrices.

## When NOT to use

- The input is already an observed ASV count/feature table — variability should come from data, not from the metaSPARSim R1-preset model.
- You need true biological replicate heterogeneity rather than MHASS's emulated model where negative-dispersion samples become identical replicates before technical sampling.
- Simulating sequencing platform noise (read errors, number of passes) rather than between-sample abundance variability — use the PBSIM3 and num-passes (empirical or lognormal, mu 3.88 / sigma 1.22) components instead.

## Inputs

- User-supplied genome molarity / relative genome abundance values (e.g. via --genome-distribution uniform, lognormal, powerlaw, or empirical:<file.tsv>)
- Amplicon FASTA file of ASV reference sequences ('amplicons.fa')
- Amplicon-to-genome label TSV mapping asvid<TAB>genomeid

## Outputs

- Genome-aware abundance matrix of ASV counts per sample with emulated between-sample variability (counts.tsv: Simulated ASV counts per sample)
- Sample library sizes (counts_meta.tsv)

## How to apply

Within the MHASS Python package's abundance-modeling code path, fit a negative binomial model using metaSPARSim's R1 preset values (variability model intercept 1.47565981333483 and slope -0.909890963463704, exposed via --var-intercept and --var-slope) to predict each ASV's squared coefficient of variation between samples as a function of its expected abundance. Draw sample-level variability per ASV from this fitted model; when the sampled squared CV would be negative, emulate it as NA, which metaSPARSim converts to 0. This yields zero biological variability (identical replicate abundances) before technical sampling noise is applied downstream. Rationale: this reproduces the empirically observed relationship in real 16S datasets where variability decreases with abundance, while avoiding negative-dispersion artifacts from the metaSPARSim fit.

## Related tools

- **MHASS** (Provides the abundance-modeling module that fits the metaSPARSim R1-preset negative binomial variability model and applies NA-to-zero dispersion emulation) — https://github.com/rhowardstone/MHASS
- **metaSPARSim** (Source of the R1 preset negative binomial model used to predict ASV squared coefficient of variation from expected abundance; also converts NA dispersion estimates to 0)
- **MHASS_evaluation** (Evaluation repository whose workflow (1A_runner.sh) validates the implemented abundance simulation module) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
mhass --amplicon-fasta amplicons.fa --amplicon-genome-labels labels.tsv --output-dir output/ --var-intercept 1.47565981333483 --var-slope -0.909890963463704
```

## Evaluation signals

- Each ASV's predicted squared coefficient of variation follows the R1-preset relationship with expected abundance (intercept 1.47565981333483, slope -0.909890963463704).
- No sampled squared CV is negative: cases that would be <0 are emulated as NA and converted to 0 by metaSPARSim, giving zero variability for those ASVs.
- ASVs assigned zero variability show identical abundances across replicates before technical sampling is applied.
- The module runs successfully as part of the full evaluation workflow (bash scripts/1A_runner.sh) after setup.sh and data downloads.
- Downstream output counts.tsv contains simulated ASV counts per sample consistent with the supplied genome molarity values.

## Limitations

- Negative sampled variability values are not modeled biologically — they are emulated as NA (converted to 0), yielding identical replicate abundances before technical sampling.
- The R1 preset fit is dataset-specific; the variability relationship may not generalize to microbiomes with different abundance-variability structure.
- Variability defaults assume the R1 fit; users must override --var-intercept/--var-slope for alternative variability profiles.

## Evidence

- [results] The abundance-modeling module fits a negative binomial model, using metaSPARSim's R1 preset values, to predict each ASV's squared coefficient of variation between samples from its expected abundance: [evidence span withheld — anchor status: not_found]
- [abstract] MHASS genome-aware abundance modeling: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] README variability model parameters: "--var-intercept VAR_INTERCEPT
                        Intercept for ASV variability model (controls baseline variation between samples) (default: 1.47565981333483)"
- [readme] README evaluation workflow validation: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:

bash
bash scripts/1A_runner.sh"
