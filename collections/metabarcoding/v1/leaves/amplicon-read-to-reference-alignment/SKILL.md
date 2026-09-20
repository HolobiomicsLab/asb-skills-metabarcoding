---
name: amplicon-read-to-reference-alignment
description: Use when real or MHASS-simulated PacBio HiFi amplicon reads must be aligned
  to reference ASV sequences to quantify per-read edit-distance distributions or position-specific
  insertion/substitution/deletion error profiles.
when_to_use_negative:
- Input is already an ASV count table or summary (e.g., MHASS counts.tsv or counts_meta.tsv)
  rather than raw reads — there is nothing to align.
- The goal is to generate simulated reads rather than measure their errors — invoke
  MHASS directly (mhass --amplicon-fasta amplicons.fa --output-dir output/) instead
  of this alignment step.
- No candidate reference ASV FASTA exists for the community — the minimum-distance-over-references
  nearest-reference metric cannot be computed.
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3307
tools:
- name: MHASS
  role: Generates the simulated HiFi amplicon reads (via PBSIM3) whose edit distances
    and per-position error profiles are quantified by this alignment step
  repo: https://github.com/rhowardstone/MHASS
- name: Python
  role: Implements the pairwise sequence alignment routine computing per-read minimum
    Levenshtein edit distances and per-position insertion/substitution/deletion counts
- name: MHASS_evaluation
  role: Provides the evaluation scripts and data (run via bash scripts/1A_runner.sh)
    reproducing the read-to-reference error analyses
  repo: https://github.com/rhowardstone/MHASS_evaluation
- name: matplotlib
  role: Plots per-position error rates versus alignment position with real vs simulated
    traces overlaid (insertion_rate_by_position.png)
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btaf656
    title: 'MHASS: Microbiome HiFi Amplicon Sequencing Simulator'
  source_content_hash: sha256:30f481544209e411a366f0bd8e71be94eb539f5e4764df2fe7ace2c188e4b86d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/rhowardstone/MHASS
  tools:
  - MHASS
  - MHASS_evaluation
  - Python
  - matplotlib
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# amplicon-read-to-reference-alignment

## Summary

Aligns real or MHASS-simulated PacBio HiFi amplicon reads against their reference ASV sequences to quantify per-read (Levenshtein) edit-distance distributions and per-position insertion/substitution/deletion error rates. It underpins MHASS validation by showing simulated error profiles mirror real data, including the Zymo/Titan-1 terminal-insertion artifact.

## When to use

Use when real or MHASS-simulated PacBio HiFi amplicon reads must be aligned to reference ASV sequences to quantify per-read edit-distance distributions or position-specific insertion/substitution/deletion error profiles.

## When NOT to use

- Input is already an ASV count table or summary (e.g., MHASS counts.tsv or counts_meta.tsv) rather than raw reads — there is nothing to align.
- The goal is to generate simulated reads rather than measure their errors — invoke MHASS directly (mhass --amplicon-fasta amplicons.fa --output-dir output/) instead of this alignment step.
- No candidate reference ASV FASTA exists for the community — the minimum-distance-over-references nearest-reference metric cannot be computed.

## Inputs

- Real PacBio HiFi amplicon read FASTQ files (Zymo/Titan-1 downloaded data file, ATCC 16S DevNet Sequel II 2.0 release, Phylotag reads from JGI)
- MHASS-simulated reads (combined_reads.fastq from each simulation output directory)
- Reference ASV/amplicon sequences in FASTA format (amplicons.fa per mock community)

## Outputs

- Per-genome per-read minimum edit-distance distributions (edit_distance_distributions.csv)
- Per-position insertion/substitution/deletion error proportions for real and simulated read sets
- insertion_rate_by_position.png figure with overlaid real vs simulated error traces

## How to apply

Load the real and MHASS-simulated HiFi read sets (e.g., the downloaded Zymo/Titan-1 amplicon data file and the simulation's combined_reads.fastq) together with their reference ASV/amplicon FASTA sequences (amplicons.fa). For each read, compute the global (Levenshtein) edit distance against every candidate reference ASV using a pairwise sequence alignment routine in Python, taking the minimum distance as the nearest-reference error, and aggregate per-read minimums into per-genome distributions per mock community. Tabulate insertion, substitution, and deletion counts at each alignment position and convert to per-position error proportions for the real and simulated Zymo/Titan-1 read sets. Repeat the same per-position computation for the 16S-only ATCC (PacificBiosciences DevNet Sequel II 2.0 release) and Phylotag (JGI) datasets as negative controls. Plot error rates versus alignment position with matplotlib, overlaying real vs simulated traces per dataset, and save as insertion_rate_by_position.png. Validate that the Zymo/Titan-1 insertion proportion rises from baseline to >90% at approximately position 2100 and stays elevated through the alignment end in both real and simulated data while the 16S-only controls show no such rise; note that simulated realism depends on PBSIM3 flags such as --subread-accuracy (default 0.65) and the platform-specific --difference-ratio.

## Related tools

- **MHASS** (Generates the simulated HiFi amplicon reads (via PBSIM3) whose edit distances and per-position error profiles are quantified by this alignment step) — https://github.com/rhowardstone/MHASS
- **Python** (Implements the pairwise sequence alignment routine computing per-read minimum Levenshtein edit distances and per-position insertion/substitution/deletion counts)
- **MHASS_evaluation** (Provides the evaluation scripts and data (run via bash scripts/1A_runner.sh) reproducing the read-to-reference error analyses) — https://github.com/rhowardstone/MHASS_evaluation
- **matplotlib** (Plots per-position error rates versus alignment position with real vs simulated traces overlaid (insertion_rate_by_position.png))

## Examples

```
bash scripts/1A_runner.sh
```

## Evaluation signals

- Per-read metric is the minimum Levenshtein edit distance over all candidate reference ASVs (nearest-reference error), aggregated per genome/mock community.
- Zymo/Titan-1: insertion proportion rises from baseline to >90% at approximately alignment position 2100 and remains elevated through the alignment end, in both real and MHASS-simulated reads.
- Negative controls: no terminal-insertion rise appears in the 16S-only ATCC/16S and Phylotag/16S datasets.
- Simulated per-genome edit-distance distributions mirror real data (bimodal for Zymo/Titan-1, unimodal for ATCC/16S, realistic low-abundance behavior for Phylotag/16S genomes).
- Deliverables produced and populated: edit_distance_distributions.csv summary table and insertion_rate_by_position.png.

## Limitations

- The >90% terminal-insertion profile is specific to the Zymo/Titan-1 amplicon; the pattern is absent in 16S-only datasets (ATCC, Phylotag) and must not be generalized to other amplicons.
- Simulated error profiles inherit PBSIM3 settings — --subread-accuracy (default 0.65) and --difference-ratio (default 6:55:39 is for PacBio RS II; 22:45:33 for Sequel, 39:24:36 for ONT) — so a platform-appropriate ratio must be chosen.
- The nearest-reference edit distance is only meaningful when the true source ASV is present among the candidate reference sequences.

## Evidence

- [abstract] MHASS generates the synthetic HiFi amplicon reads analyzed by this skill: "Microbiome HiFi Amplicon Sequence Simulator (MHASS) creates realistic synthetic PacBio HiFi amplicon sequencing datasets for microbiome studies"
- [methods] ATCC 16S reference dataset source: "Then download the ATCC dataset: https://github.com/PacificBiosciences/DevNet/wiki/16S-Data-Set-Sequel-II-System-2.0-Release"
- [methods] Phylotag 16S reference dataset source: "And the Phylotag dataset: https://genome.jgi.doe.gov/portal/PhyloTag/PhyloTag.home.html"
- [methods] Per-read nearest-reference edit distance method: [evidence span withheld — anchor status: not_found]
- [results] Zymo/Titan-1 terminal-insertion validation pattern: [evidence span withheld — anchor status: not_found]
- [readme] PBSIM3 subread accuracy parameter: [evidence span withheld — anchor status: external_doc]
- [readme] Platform-specific difference ratio for PBSIM: [evidence span withheld — anchor status: external_doc]
- [readme] Simulated reads output file consumed by the alignment: [evidence span withheld — anchor status: external_doc]
