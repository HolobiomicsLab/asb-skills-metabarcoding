---
name: distribution-shape-comparison-across-datasets
description: Use when simulated amplicon reads must be validated against real HiFi
  data by comparing per-genome error (edit distance) distributions and their shape
  (bimodal vs unimodal) across mock communities.
when_to_use_negative:
- Input is already a feature/ASV count table with no read-level sequences — edit distances
  cannot be computed
- Reads are short Illumina amplicon reads rather than PacBio HiFi long-read amplicon
  data; the error model and distributions being compared are HiFi-specific
edam_topics:
- http://edamontology.org/topic_0196
- http://edamontology.org/topic_3174
tools:
- name: MHASS
  role: Generator of the simulated HiFi amplicon reads whose edit distance distributions
    are compared against real data
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Used to implement the pairwise sequence alignment routine that computes global
    (Levenshtein) edit distances between simulated reads and reference ASVs
- name: bash
  role: Runs the evaluation pipeline (setup and runner scripts) that produces the
    simulation and analysis outputs
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:75009190f19cae26ee4addf3efb37940fd9706f8bf5dfd0126f5c6745000c9d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - Python
  - bash
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0196
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# distribution-shape-comparison-across-datasets

## Summary

Validates a microbiome HiFi sequencing simulator (MHASS) by comparing per-genome edit distance distributions of simulated PacBio HiFi amplicon reads against real sequencing data across three mock communities (Zymo/Titan-1, ATCC/16S, Phylotag/16S).

## When to use

Use when simulated amplicon reads must be validated against real HiFi data by comparing per-genome error (edit distance) distributions and their shape (bimodal vs unimodal) across mock communities.

## When NOT to use

- Input is already a feature/ASV count table with no read-level sequences — edit distances cannot be computed
- Reads are short Illumina amplicon reads rather than PacBio HiFi long-read amplicon data; the error model and distributions being compared are HiFi-specific

## Inputs

- MHASS-simulated PacBio HiFi amplicon reads (combined_reads.fastq) for Zymo/Titan-1, ATCC/16S, and Phylotag/16S mock communities
- Reference ASV sequences (amplicon FASTA, e.g. amplicons.fa) for each mock community
- Real sequencing data per-genome edit distance distributions from Fig. 1 for comparison

## Outputs

- Per-genome edit distance distribution summary table (edit_distance_distributions.csv)
- Shape-match assessment per mock community (bimodal Zymo/Titan-1, unimodal ATCC/16S, low-abundance Phylotag/16S)

## How to apply

Load the MHASS-simulated HiFi reads for each mock community (Zymo/Titan-1, ATCC/16S from the PacificBiosciences DevNet 16S Sequel II System 2.0 Release, and Phylotag/16S from JGI) together with the reference ASV sequences (amplicons.fa). For each simulated read, compute the global (Levenshtein) edit distance against every candidate reference ASV using a pairwise sequence alignment routine in Python, taking the minimum distance as the nearest-reference error. Aggregate the per-read minimum edit distances into per-genome distributions for each mock community. Then compare each simulated per-genome distribution against the real-data distributions (Fig. 1), checking for the expected shape: bimodal for Zymo/Titan-1, unimodal for ATCC/16S, and realistic low-abundance behavior for Phylotag/16S genomes, and write the summary table to edit_distance_distributions.csv.

## Related tools

- **MHASS** (Generator of the simulated HiFi amplicon reads whose edit distance distributions are compared against real data) — https://github.com/rhowardstone/MHASS
- **Python** (Used to implement the pairwise sequence alignment routine that computes global (Levenshtein) edit distances between simulated reads and reference ASVs)
- **bash** (Runs the evaluation pipeline (setup and runner scripts) that produces the simulation and analysis outputs)

## Examples

```
git clone https://github.com/rhowardstone/MHASS_evaluation.git; cd MHASS_evaluation; bash scripts/1A_runner.sh
```

## Evaluation signals

- edit_distance_distributions.csv exists and contains per-genome distribution summaries for all three mock communities
- Zymo/Titan-1 simulated distribution is bimodal, matching the real-data pattern in Fig. 1
- ATCC/16S simulated distribution is unimodal, matching the real-data pattern
- Phylotag/16S genomes show realistic low-abundance behavior in the simulated distributions
- Per-read minimum edit distances are non-negative integers and each read is assigned the minimum distance over all candidate reference ASVs

## Limitations

- The comparison relies on pairwise alignment of every read against every reference ASV, which scales with read count times ASV count
- Results depend on the PBSIM error model parameters (e.g. --difference-ratio and --subread-accuracy) chosen during simulation; default 6:55:39 is for PacBio RS II rather than Sequel II
- Fidelity is judged by distribution shape agreement with Fig. 1 patterns; deviations could arise from reference set incompleteness rather than simulator error

## Evidence

- [methods] For each simulated read, compute the global (Levenshtein) edit distance against every candidate reference ASV using a pairwise sequence alignment routine in Python, taking the minimum distance as the nearest-reference error.: [evidence span withheld — anchor status: not_found]
- [results] Across all three mock communities, MHASS generated reads with edit distance distributions that closely mirrored those observed in real sequencing data.: "Across all three mock communities, MHASS generated reads with edit distance distributions that closely mirrored those observed in real sequencing data."
- [abstract] Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [readme] Download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
