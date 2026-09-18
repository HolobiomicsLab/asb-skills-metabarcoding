---
name: amplicon-sequencing-simulation
description: Use when you need synthetic PacBio HiFi amplicon data (e.g. 16S reads)
  from known genomes to benchmark ASV clustering or taxonomic assignment, or to compare
  simulated vs real mock-community abundance correlations.
when_to_use_negative:
- You need simulated reads for shotgun metagenomics or short-read (Illumina) platforms
  — MHASS models PacBio HiFi amplicon data with PBSIM3 error profiles
- Input is already an ASV feature table and the goal is statistical analysis rather
  than generating synthetic sequencing data
- No genome or amplicon reference sequences are available, since MHASS requires an
  amplicon FASTA and genome labels
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: MHASS
  role: Core simulator producing synthetic PacBio HiFi amplicon FASTQ and count tables
    from genome-aware abundance models
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implementation language of MHASS; used with automated dependency management
- name: AmpliconHunter
  role: Generates the ASV FASTA (amplicons.fa) required as MHASS input from a collection
    of genomes
  repo: https://github.com/rhowardstone/AmpliconHunter
- name: git
  role: Clones the MHASS and MHASS_evaluation repositories during setup
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: bash
  role: Runs the installation and evaluation runner scripts (install_dependencies.sh,
    setup.sh, 1A_runner.sh)
merged_aliases:
- amplicon-sequence-simulation-execution
merged_alias_records:
- alias: amplicon-sequence-simulation-execution
  slug: amplicon-sequence-simulation-execution
  cosine_score: 0.8919
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:195347b355f1796c6709f7f3c922c7a1c13ecaa5edc8fbfe1b7f7547281beb3d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - AmpliconHunter
  - MHASS
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# amplicon-sequencing-simulation

## Summary

Simulate realistic PacBio HiFi amplicon sequencing datasets for microbiome studies using MHASS, which integrates genome-aware abundance modeling, dual-barcoding, and empirically derived number-of-passes distributions, enabling benchmarking of long-read workflows such as ASV clustering and taxonomic assignment.

## When to use

Use when you need synthetic PacBio HiFi amplicon data (e.g. 16S reads) from known genomes to benchmark ASV clustering or taxonomic assignment, or to compare simulated vs real mock-community abundance correlations.

## When NOT to use

- You need simulated reads for shotgun metagenomics or short-read (Illumina) platforms — MHASS models PacBio HiFi amplicon data with PBSIM3 error profiles
- Input is already an ASV feature table and the goal is statistical analysis rather than generating synthetic sequencing data
- No genome or amplicon reference sequences are available, since MHASS requires an amplicon FASTA and genome labels

## Inputs

- Amplicon reference FASTA (amplicons.fa) with one ASV sequence per record
- Amplicon-to-genome label TSV (asvid<TAB>genomeid)
- Optional dual-barcode TSV with columns id, forward, reverse
- Empirical number-of-passes distribution TSV (required when --np-distribution-type empirical)
- Optional genome abundance distribution file (empirical:<file>)

## Outputs

- combined_reads.fastq (final simulated HiFi read FASTQ)
- counts.tsv (simulated ASV counts per sample)
- counts_meta.tsv (sample library sizes)
- sample_barcode_map.tsv (sample-to-barcode mapping)
- sequence_file_mapping.tsv (detailed template mappings)

## How to apply

Clone the MHASS repository and install dependencies (conda, samtools) via 'bash install_dependencies.sh', then 'conda activate mhass'. Build an amplicon FASTA (amplicons.fa) from genomes — e.g. with AmpliconHunter — plus a TSV mapping each ASV to a genome ID (asvid<TAB>genomeid), and optionally a barcode TSV (id, forward, reverse) for dual-barcoding. Choose a genome abundance distribution (uniform, lognormal, powerlaw, or empirical:<file>), a number-of-passes distribution (empirical TSV or lognormal with defaults mu=3.88, sigma=1.22, np-min=2, np-max=59), subread accuracy (default 0.65), and a PBSIM difference-ratio appropriate to the platform (default 6:55:39 for PacBio RS II). Set --num-samples and --num-reads to match the target study scale; simulation time and memory scale linearly with read count plus overhead from empirical pass-number sampling. Outputs include counts.tsv (ASV counts per sample), counts_meta.tsv (library sizes), sample_barcode_map.tsv, sequence_file_mapping.tsv, and combined_reads.fastq; validate by comparing expected vs observed ASV abundances (Pearson R2) against real mock-community data.

## Related tools

- **MHASS** (Core simulator producing synthetic PacBio HiFi amplicon FASTQ and count tables from genome-aware abundance models) — https://github.com/rhowardstone/MHASS
- **Python** (Implementation language of MHASS; used with automated dependency management)
- **AmpliconHunter** (Generates the ASV FASTA (amplicons.fa) required as MHASS input from a collection of genomes) — https://github.com/rhowardstone/AmpliconHunter
- **git** (Clones the MHASS and MHASS_evaluation repositories during setup) — https://github.com/rhowardstone/MHASS_evaluation
- **bash** (Runs the installation and evaluation runner scripts (install_dependencies.sh, setup.sh, 1A_runner.sh))

## Examples

```
mhass --amplicon-fasta amplicons.fa --amplicon-genome-labels labels.tsv --barcode-file barcodes.tsv --np-distribution-type lognormal --output-dir testout
```

## Evaluation signals

- Pearson R2 between expected and observed ASV abundances should approximate real mock-community values, e.g. 0.519 vs 0.662 (real) for Zymo/Titan-1, 0.999 vs 0.956 for ATCC/16S, and 0.944 vs 0.794 for Phylotag/16S
- The output directory contains all five expected files: counts.tsv, counts_meta.tsv, sample_barcode_map.tsv, sequence_file_mapping.tsv, combined_reads.fastq
- counts_meta.tsv library sizes and per-sample read counts match the requested --num-samples and --num-reads settings
- Wall-clock runtime scales linearly with read count, consistent with reported benchmarks (40 min for ~2k reads/sample across 96 samples; 6.75 h for ~13k reads/sample across 192 samples; 23 min for ~23k reads/sample across 5 samples)
- Sequence error profile in simulated reads reflects the chosen --difference-ratio and --subread-accuracy (default 6:55:39 and 0.65) rather than other platform profiles

## Limitations

- Installation requires conda and samtools
- Number-of-passes distributions other than lognormal require an empirical TSV file; empirical sampling adds overhead beyond linear read-count scaling of time and memory
- The default --difference-ratio 6:55:39 is for PacBio RS II and must be changed (e.g. 22:45:33 for Sequel, 39:24:36 for ONT) for other platforms
- For evaluation reproductions, runner script path references must be manually updated to the locations of the downloaded supplementary data, ATCC dataset, and Phylotag dataset

## Evidence

- [abstract] Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [abstract] MHASS generates datasets tailored for rigorous benchmarking and validation of long-read microbiome analysis workflows, including ASV clustering and taxonomic assignment: "MHASS generates datasets tailored for rigorous benchmarking and validation of long-read microbiome analysis workflows, including ASV clustering and taxonomic assignment"
- [methods] git clone https://github.com/rhowardstone/MHASS_evaluation.git: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [readme] Difference (error) ratio for PBSIM in format `substitution:insertion:deletion`. Each value must be 0-1000. Default `6:55:39` is for PacBio RS II.: [evidence span withheld — anchor status: external_doc]
- [readme] This will produce a directory at `./testout`, containing the output of the simulation: * `counts.tsv`: Simulated ASV counts per sample * `counts_meta.tsv`: Sample library sizes: [evidence span withheld — anchor status: external_doc]
- [results] Expected and observed ASV abundance correlations were: Zymo/Titan-1 (even abundances): R2=0.662 (real), R2=0.519 (simulated); ATCC/16S (staggered): R2=0.956 (real), R2=0.999 (simulated).: [evidence span withheld — anchor status: not_found]
