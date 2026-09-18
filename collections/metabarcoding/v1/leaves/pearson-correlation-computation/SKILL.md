---
name: pearson-correlation-computation
description: Use when you have mock-community amplicon data with known expected genome/ASV
  abundances (e.g., Zymo, ATCC 16S, Phylotag) and simulated or real observed counts,
  and need R2 agreement metrics.
when_to_use_negative:
- Input is already a curated abundance table and the goal is plotting or taxonomy
  rather than expected-vs-observed agreement quantification
- The community lacks ground-truth expected abundances (no known mock composition),
  so a Pearson R2 against expected values cannot be computed
- The goal is generating new simulated reads rather than evaluating an existing simulation
  — use MHASS simulator skills instead
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Python
  role: Compute Pearson R2 values between expected and observed ASV/genome abundances
    and assemble results into 1A_r2_results.csv
- name: MHASS
  role: Generates the simulated PacBio HiFi amplicon datasets whose observed ASV abundances
    are correlated against expected abundances
  repo: https://github.com/rhowardstone/MHASS
- name: bash
  role: Runs scripts/setup.sh for installation and scripts/1A_runner.sh to launch
    the three mock-community simulations
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:8b905eca1002aaa40fda99f0d1ace98bcbdcfd8d3a043a7960bdcc6189a75b97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  - git
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# pearson-correlation-computation

## Summary

Computes Pearson R2 between expected and observed ASV (or genome-level) abundances to quantify how faithfully MHASS-simulated PacBio HiFi amplicon datasets reproduce real mock-community sequencing, as reported for Zymo/Titan-1, ATCC/16S, and Phylotag/16S datasets.

## When to use

Use when you have mock-community amplicon data with known expected genome/ASV abundances (e.g., Zymo, ATCC 16S, Phylotag) and simulated or real observed counts, and need R2 agreement metrics.

## When NOT to use

- Input is already a curated abundance table and the goal is plotting or taxonomy rather than expected-vs-observed agreement quantification
- The community lacks ground-truth expected abundances (no known mock composition), so a Pearson R2 against expected values cannot be computed
- The goal is generating new simulated reads rather than evaluating an existing simulation — use MHASS simulator skills instead

## Inputs

- MHASS simulator outputs: counts.tsv (simulated ASV counts per sample), counts_meta.tsv (sample library sizes)
- Expected genome/ASV abundance tables for the mock communities (Zymo/Titan-1, ATCC/16S, Phylotag/16S)
- Real PacBio HiFi mock-community sequencing data (ATCC 16S Data Set Sequel II System 2.0 Release; Phylotag dataset)

## Outputs

- 1A_r2_results.csv with six Pearson R2 values (real and simulated, at genome level and ASV level, for three mock communities)
- Per-dataset expected-vs-observed abundance comparison tables

## How to apply

After installing the MHASS_evaluation repository (git clone, then 'bash scripts/setup.sh') and downloading the MHASS data file, the ATCC 16S Data Set (Sequel II System 2.0 Release from PacificBiosciences DevNet), and the Phylotag dataset, edit the runner scripts so path references point at your downloads. Run 'bash scripts/1A_runner.sh' so MHASS simulates the Zymo/Titan-1, ATCC/16S, and Phylotag/16S mock communities; each simulation writes outputs such as counts.tsv (simulated ASV counts per sample) and counts_meta.tsv (sample library sizes). Parse the simulator outputs together with expected abundances and compute Pearson R2 with Python at both genome level and ASV level for each of the three datasets, producing six values. Assemble the values into 1A_r2_results.csv. Validate by comparing simulated R2 against the reported real-data values: Zymo/Titan-1 (even abundances) 0.662 real vs 0.519 simulated; ATCC/16S (staggered) 0.956 real vs 0.999 simulated; Phylotag/16S 0.794 real vs 0.944.

## Related tools

- **Python** (Compute Pearson R2 values between expected and observed ASV/genome abundances and assemble results into 1A_r2_results.csv)
- **MHASS** (Generates the simulated PacBio HiFi amplicon datasets whose observed ASV abundances are correlated against expected abundances) — https://github.com/rhowardstone/MHASS
- **bash** (Runs scripts/setup.sh for installation and scripts/1A_runner.sh to launch the three mock-community simulations)

## Examples

```
bash scripts/1A_runner.sh
```

## Evaluation signals

- Computed R2 values match the reported real-data correlations: 0.662 (Zymo/Titan-1 real), 0.956 (ATCC/16S real), 0.794 (Phylotag/16S real)
- Simulated R2 values fall in the reported ranges: 0.519 (Zymo/Titan-1), 0.999 (ATCC/16S), 0.944 (Phylotag/16S)
- 1A_r2_results.csv contains exactly six values covering three datasets at genome and ASV levels
- Simulator outputs parse cleanly: counts.tsv provides per-sample ASV counts and counts_meta.tsv provides library sizes
- Simulated R2 should approximate (not necessarily match) real R2, with even communities (Zymo/Titan-1) showing the largest gap

## Limitations

- Simulated R2 can exceed real R2 (ATCC/16S: 0.999 simulated vs 0.956 real), so higher correlation does not imply closer realism
- The even-abundance Zymo/Titan-1 community shows the largest real-to-simulated R2 degradation (0.662 to 0.519), so agreement is community-composition dependent
- Runner script path references must be manually edited to reflect download locations, and MHASS installation requires conda and samtools

## Evidence

- [results] Expected and observed ASV abundance correlations were: Zymo/Titan-1 (even abundances): R2=0.662 (real), R2=0.519 (simulated); ATCC/16S (staggered): R2=0.956 (real), R2=0.999 (simulated).: [evidence span withheld — anchor status: not_found]
- [methods] Parse the simulator outputs and compare expected versus observed ASV abundances at genome level and ASV level for each of the three datasets.: [evidence span withheld — anchor status: not_found]
- [readme] After installation, update the references in the runner scripts to reflect the location of your downloads, and run: bash scripts/1A_runner.sh: "After installation, update the references in the runner scripts to reflect the location of your downloads, and run:

bash
bash scripts/1A_runner.sh"
- [abstract] MHASS creates realistic synthetic PacBio HiFi amplicon sequencing datasets by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions: "by integrating genome-aware abundance modeling, realistic dual-barcoding strategies, and empirically derived pass-number distributions from actual sequencing runs"
- [readme] counts.tsv: Simulated ASV counts per sample: "* `counts.tsv`: Simulated ASV counts per sample
* `counts_meta.tsv`: Sample library sizes"
