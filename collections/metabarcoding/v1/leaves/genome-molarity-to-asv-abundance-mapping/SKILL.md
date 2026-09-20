---
name: genome-molarity-to-asv-abundance-mapping
description: Use when simulating PacBio HiFi amplicon datasets and you need to turn
  genome-level abundance/molarity inputs into realistic per-sample ASV count matrices
  with metaSPARSim-style negative binomial variability.
when_to_use_negative:
- Input is already an ASV feature/count table — no genome-to-amplicon molarity mapping
  is needed or available
- Simulating Illumina short-read or single-end amplicon data — MHASS targets PacBio
  HiFi amplicon sequencing with dual barcoding and pass-number (np) distributions
- You only need technical read-level simulation (PBSIM error model, barcoding) without
  genome-aware biological abundance variability
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3334
tools:
- name: MHASS
  role: Python simulator whose abundance-modeling module performs the genome molarity
    to ASV abundance mapping with metaSPARSim-based negative binomial variability
  repo: https://github.com/rhowardstone/MHASS
- name: metaSPARSim
  role: Source of the R1 preset negative binomial parameters (var-intercept/var-slope)
    used to predict per-ASV between-sample squared coefficient of variation; also
    converts NA dispersion estimates to 0
- name: Python
  role: Implementation language for the abundance-modeling code path and automated
    dependency management
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:814e7145e5f802842ca5b317750488bc50074943a4a1f11e93a821f9de21b79e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - git
  - metaSPARSim
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3334
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# genome-molarity-to-asv-abundance-mapping

## Summary

MHASS's genome-aware abundance simulation step converts user-supplied genome molarity values into per-sample ASV abundance matrices by fitting a negative binomial model with metaSPARSim's R1 preset parameters to model between-sample variability for each ASV. Negative sampled variability estimates are emulated as NA (converted to 0 by metaSPARSim), yielding identical replicate abundances before technical sampling.

## When to use

Use when simulating PacBio HiFi amplicon datasets and you need to turn genome-level abundance/molarity inputs into realistic per-sample ASV count matrices with metaSPARSim-style negative binomial variability.

## When NOT to use

- Input is already an ASV feature/count table — no genome-to-amplicon molarity mapping is needed or available
- Simulating Illumina short-read or single-end amplicon data — MHASS targets PacBio HiFi amplicon sequencing with dual barcoding and pass-number (np) distributions
- You only need technical read-level simulation (PBSIM error model, barcoding) without genome-aware biological abundance variability

## Inputs

- User-supplied genome molarity values (genome-level abundances)
- Amplicon FASTA file of ASV reference sequences (amplicons.fa)
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- Optional genome abundance distribution file for --genome-distribution empirical:<file>

## Outputs

- Simulated ASV counts per sample (counts.tsv)
- Sample library sizes (counts_meta.tsv)
- Genome-level abundance matrix produced by metaSPARSim-driven simulation before technical sampling

## How to apply

In the MHASS Python package, take user-supplied genome molarity values and draw relative genome abundances from the chosen distribution (--genome-distribution: uniform, lognormal, powerlaw, or empirical:<file>). Map genomes to ASVs using the amplicon-to-genome label TSV (asvid<TAB>genomeid) so genome-level abundances aggregate to the ASV level. Fit a negative binomial model with metaSPARSim's R1 preset values — var-intercept 1.47565981333483 and var-slope -0.909890963463704 — to predict each ASV's squared coefficient of variation between samples from its expected abundance. When the sampled variability would be < 0, write NA, which metaSPARSim converts to 0, producing zero biological variability (identical replicate abundances) before technical sampling is applied. Output the simulated ASV counts per sample (counts.tsv) and library sizes (counts_meta.tsv).

## Related tools

- **MHASS** (Python simulator whose abundance-modeling module performs the genome molarity to ASV abundance mapping with metaSPARSim-based negative binomial variability) — https://github.com/rhowardstone/MHASS
- **metaSPARSim** (Source of the R1 preset negative binomial parameters (var-intercept/var-slope) used to predict per-ASV between-sample squared coefficient of variation; also converts NA dispersion estimates to 0)
- **Python** (Implementation language for the abundance-modeling code path and automated dependency management)

## Examples

```
mhass --amplicon-fasta Test-data_ATCC_16S_MSA-1003/amplicons.fa --amplicon-genome-labels Test-data_ATCC_16S_MSA-1003/amplicon_labels.txt --output-dir testout
```

## Evaluation signals

- counts.tsv contains per-sample ASV counts whose between-sample variability matches the negative binomial model with var-intercept 1.47565981333483 and var-slope -0.909890963463704 (R1 preset)
- Any ASV whose predicted squared coefficient of variation samples to a value < 0 yields identical abundances across biological replicates before technical sampling (NA-to-zero emulation)
- Per-sample library sizes in counts_meta.tsv sum to the configured --num-reads per sample
- ASV-level abundances aggregate correctly from genome molarity via the amplicon-genome-labels mapping
- The module passes validation by executing bash scripts/1A_runner.sh in MHASS_evaluation after updating download references in the runner scripts

## Limitations

- When the negative binomial model predicts variability < 0 it is emulated as NA and treated as 0, so such ASVs show no biological between-sample variability before technical sampling
- Variability model defaults are fixed to the metaSPARSim R1 preset; other presets would require overriding --var-intercept and --var-slope
- Requires conda and samtools for installation, and a valid amplicon-to-genome label file; ASV FASTA generation from genomes needs the separate AmpliconHunter tool

## Evidence

- [results] The abundance-modeling module fits a negative binomial model, using metaSPARSim's R1 preset values, to predict each ASV's squared coefficient of variation between samples from its expected abundance: [evidence span withheld — anchor status: not_found]
- [results] when the sampled variability would be <0 it is emulated as NA, which metaSPARSim converts to 0, yielding no biological variability (identical replicate abundances before technical sampling): [evidence span withheld — anchor status: not_found]
- [readme] README var-intercept parameter description with R1 preset default: "--var-intercept VAR_INTERCEPT
                        Intercept for ASV variability model (controls baseline variation between samples)
                            (default: 1.47565981333483)"
- [readme] README genome distribution options for genome abundances: "--genome-distribution GENOME_DISTRIBUTION
                        Distribution for genome abundances (uniform, lognormal, powerlaw, or empirical:<file>)
                            (default: uniform)"
- [abstract] abstract on genome-aware abundance modeling as part of MHASS's design: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
