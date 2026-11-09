---
name: edit-distance-distribution-computation
description: Use when benchmarking a HiFi amplicon simulator (e.g., MHASS) by checking
  whether per-genome nearest-reference edit distance distributions of simulated reads
  mirror real-data distributions for mock communities like Zymo/Titan-1, ATCC/16S,
  or Phylotag/16S.
when_to_use_negative:
- Input is already a per-read edit distance table or distribution summary — skip to
  the comparison step instead of recomputing distances
- The goal is taxonomic assignment or ASV clustering of reads rather than benchmarking
  simulator error profiles
- No reference ASV FASTA exists for the mock community, so nearest-reference distances
  cannot be defined
tools:
- name: MHASS
  role: Source of the simulated PacBio HiFi amplicon reads whose edit distance distributions
    are computed
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implements the pairwise sequence alignment routine used to compute global
    (Levenshtein) edit distances between reads and reference ASVs
- name: MHASS_evaluation
  role: Evaluation repository providing scripts and data for the analyses, including
    the edit distance distribution comparison
  repo: https://github.com/rhowardstone/MHASS_evaluation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:242e198916b7937465efdccf68662d0d77d0f9d5165a7a46470f8dfe08816b7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - MHASS_evaluation
  - Python
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# edit-distance-distribution-computation

## Summary

Computes per-genome distributions of minimum (Levenshtein) edit distances between simulated or real PacBio HiFi amplicon reads and their candidate reference ASV sequences, enabling direct comparison of error profiles between MHASS-simulated and real sequencing data across mock communities.

## When to use

Use when benchmarking a HiFi amplicon simulator (e.g., MHASS) by checking whether per-genome nearest-reference edit distance distributions of simulated reads mirror real-data distributions for mock communities like Zymo/Titan-1, ATCC/16S, or Phylotag/16S.

## When NOT to use

- Input is already a per-read edit distance table or distribution summary — skip to the comparison step instead of recomputing distances
- The goal is taxonomic assignment or ASV clustering of reads rather than benchmarking simulator error profiles
- No reference ASV FASTA exists for the mock community, so nearest-reference distances cannot be defined

## Inputs

- MHASS-simulated HiFi read FASTQ files (combined_reads.fastq) for Zymo/Titan-1, ATCC/16S, and Phylotag/16S mock communities
- Reference ASV FASTA sequences (amplicons.fa) for each mock community
- Real sequencing data edit distance distributions from the paper (Fig. 1) for comparison

## Outputs

- edit_distance_distributions.csv (per-genome distribution summary table)
- Per-read minimum edit distance values against the nearest reference ASV
- Comparison of simulated vs. real per-genome edit distance distribution shapes (bimodal/unimodal/low-abundance)

## How to apply

Load the MHASS-simulated HiFi FASTQ reads together with the reference ASV FASTA sequences for each mock community (ATCC from the PacificBiosciences DevNet 16S Data Set Sequel II System 2.0 Release; Phylotag from JGI). For each simulated read, compute the global (Levenshtein) edit distance against every candidate reference ASV using a pairwise sequence alignment routine in Python, taking the minimum distance as the nearest-reference error. Aggregate per-read minimum edit distances into per-genome distributions for each community, then compare each simulated distribution against the real-data distributions reported in Fig. 1, checking the expected patterns: bimodal for Zymo/Titan-1, unimodal for ATCC/16S, and realistic low-abundance behavior for Phylotag/16S genomes. Write the resulting per-genome distribution summary to edit_distance_distributions.csv.

## Related tools

- **MHASS** (Source of the simulated PacBio HiFi amplicon reads whose edit distance distributions are computed) — https://github.com/rhowardstone/MHASS
- **Python** (Implements the pairwise sequence alignment routine used to compute global (Levenshtein) edit distances between reads and reference ASVs)
- **MHASS_evaluation** (Evaluation repository providing scripts and data for the analyses, including the edit distance distribution comparison) — https://github.com/rhowardstone/MHASS_evaluation

## Examples

```
from Levenshtein import distance; min_d = min(distance(read_seq, asv_seq) for asv_seq in asv_seqs)  # per-read nearest-reference edit distance
```

## Evaluation signals

- edit_distance_distributions.csv is produced with per-genome summary rows for each of the three mock communities
- Simulated Zymo/Titan-1 per-genome edit distance distributions are bimodal, matching the real-data pattern in Fig. 1
- Simulated ATCC/16S per-genome distributions are unimodal, matching the real-data pattern
- Phylotag/16S genomes show realistic low-abundance behavior in their edit distance distributions
- Every simulated read has a defined minimum edit distance against at least one candidate reference ASV

## Limitations

- Distances are defined relative to available reference ASVs; reads from genomes lacking a close reference ASV will show inflated minimum edit distances
- For Phylotag/16S, low-abundance genomes yield few reads, making their per-genome distributions sparse and harder to compare against real data
- The comparison requires the real-data distributions from Fig. 1 as a reference point; without them, only internal consistency of the simulated distributions can be checked

## Evidence

- [abstract] Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [methods] Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [readme] Evaluation scripts and data for all analyses in our evaluation of MHASS (https://github.com/rhowardstone/MHASS): "Evaluation scripts and data for all analyses in our evaluation of MHASS"
