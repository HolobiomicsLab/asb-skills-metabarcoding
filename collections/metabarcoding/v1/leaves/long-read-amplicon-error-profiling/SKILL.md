---
name: long-read-amplicon-error-profiling
description: Use when validating that simulated HiFi amplicon reads reproduce the
  error (edit distance) distributions of real sequencing data for mock communities
  such as Zymo/Titan-1, ATCC/16S, or Phylotag/16S.
when_to_use_negative:
- The goal is to simulate reads rather than profile errors of existing reads — use
  MHASS directly instead
- Input reads are short-read (Illumina) amplicon data; the error model and HiFi pass-number
  logic here target PacBio HiFi circular consensus reads
- Only a single genome/ASV is present, so a per-genome distribution comparison against
  real mock-community data is not meaningful
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3697
tools:
- name: MHASS
  role: Generates the simulated PacBio HiFi amplicon reads whose edit-distance distributions
    are profiled and compared against real data
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implements the pairwise sequence alignment routine computing global (Levenshtein)
    edit distances between reads and reference ASVs
- name: MHASS_evaluation
  role: Provides the evaluation scripts, runner (1A_runner.sh), and data underlying
    the edit distance distribution analyses
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:40a8f3ef6fc36e80b0d159047af113a1fe006fb68a239abf1005832730316437
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - MHASS_evaluation
  - Python
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# long-read-amplicon-error-profiling

## Summary

Profiles per-genome edit-distance distributions of PacBio HiFi amplicon reads (simulated by MHASS or real) by computing the minimum Levenshtein distance from each read to candidate reference ASVs, enabling validation that simulated error profiles match real sequencing data across mock communities.

## When to use

Use when validating that simulated HiFi amplicon reads reproduce the error (edit distance) distributions of real sequencing data for mock communities such as Zymo/Titan-1, ATCC/16S, or Phylotag/16S.

## When NOT to use

- The goal is to simulate reads rather than profile errors of existing reads — use MHASS directly instead
- Input reads are short-read (Illumina) amplicon data; the error model and HiFi pass-number logic here target PacBio HiFi circular consensus reads
- Only a single genome/ASV is present, so a per-genome distribution comparison against real mock-community data is not meaningful

## Inputs

- MHASS-simulated HiFi amplicon reads in FASTQ format (e.g., combined_reads.fastq from MHASS output directories)
- Reference ASV sequences in FASTA format (amplicons.fa, e.g., Test-data_ATCC_16S_MSA-1003/amplicons.fa)
- Amplicon-to-genome mapping TSV (asvid<TAB>genomeid) to assign per-genome distributions
- Real sequencing data distributions for comparison (ATCC DevNet 16S Sequel II 2.0 release; Phylotag dataset from JGI)

## Outputs

- Per-read minimum Levenshtein edit distances to the nearest reference ASV
- Per-genome edit distance distributions for each mock community
- Summary table edit_distance_distributions.csv

## How to apply

Load the MHASS-simulated HiFi reads (combined_reads.fastq output) for each mock community together with the corresponding reference ASV sequences (ATCC dataset from the PacificBiosciences DevNet 16S Data Set Sequel II System 2.0 Release; Phylotag dataset from JGI). For each simulated read, compute the global (Levenshtein) edit distance against every candidate reference ASV using a pairwise sequence alignment routine in Python, taking the minimum distance as the nearest-reference error. Aggregate the per-read minimum edit distances into per-genome distributions for each community. Compare each simulated per-genome distribution against the real-data distributions reported in Fig. 1, checking for the expected patterns: bimodal for Zymo/Titan-1, unimodal for ATCC/16S, and realistic low-abundance behavior for Phylotag/16S genomes.

## Related tools

- **MHASS** (Generates the simulated PacBio HiFi amplicon reads whose edit-distance distributions are profiled and compared against real data) — https://github.com/rhowardstone/MHASS
- **Python** (Implements the pairwise sequence alignment routine computing global (Levenshtein) edit distances between reads and reference ASVs)
- **MHASS_evaluation** (Provides the evaluation scripts, runner (1A_runner.sh), and data underlying the edit distance distribution analyses) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git; cd MHASS_evaluation; bash scripts/1A_runner.sh
```

## Evaluation signals

- Simulated per-genome edit distance distribution for Zymo/Titan-1 is bimodal, matching the real-data pattern in Fig. 1
- Simulated per-genome edit distance distribution for ATCC/16S is unimodal, matching real data
- Phylotag/16S genomes show realistic low-abundance behavior in their simulated edit distance distributions
- edit_distance_distributions.csv is produced with per-genome distribution summaries for all three mock communities
- Every simulated read has a well-defined minimum edit distance (non-negative integer) against the candidate ASV set

## Limitations

- Edit distance is computed against a fixed set of candidate reference ASVs; reads from genomes absent from the reference set inflate the minimum distance
- The comparison relies on real-data distributions from specific sources (ATCC DevNet 16S Sequel II 2.0 release, JGI Phylotag); other platforms/runs may show different patterns
- MHASS read generation depends on the number-of-passes distribution (empirical or lognormal, np 2–59 by default) and PBSIM3 error ratios; misconfigured parameters distort the resulting edit distance profile

## Evidence

- [other] For each simulated read, compute the global (Levenshtein) edit distance against every candidate reference ASV using a pairwise sequence alignment routine in Python: [evidence span withheld — anchor status: not_found]
- [other] Finding on matching distributions: "Across all three mock communities, MHASS generated reads with edit distance distributions that closely mirrored those observed in real sequencing data."
- [abstract] MHASS creates realistic synthetic PacBio HiFi amplicon sequencing datasets: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [methods] Evaluation repo installation and data download workflow: "git clone https://github.com/rhowardstone/MHASS_evaluation.git"
- [methods] ATCC dataset source: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [readme] MHASS output FASTQ file: [evidence span withheld — anchor status: external_doc]
