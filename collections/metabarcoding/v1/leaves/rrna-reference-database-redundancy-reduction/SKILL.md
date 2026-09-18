---
name: rrna-reference-database-redundancy-reduction
description: 'Use when designing or evaluating a reduced-redundancy rRNA reference
  database: you need to know how removing near-identical full-length sequences affects
  mapping/classification accuracy (F½ score) of MAPseq or competing tools.'
when_to_use_negative:
- Input is already a MAPseq .mseq output or an OTU/taxa count table (mapseq -otucounts
  / -otutable stage) — this skill concerns reference database construction, not post-hoc
  summarization
- The benchmark reads lack ground-truth assignments, since F½ accuracy against known
  identities cannot be computed
- The goal is a pure speed benchmark rather than quantifying the accuracy cost of
  reference redundancy reduction
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: MAPseq 1.2.6
  role: k-mer search classifier used to map known-identity rRNA reads against each
    redundancy-reduced reference and produce confidence-annotated taxonomy/OTU assignments
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: generates the 'configure' script used to build MAPseq from the cloned repository
    source
  repo: https://github.com/jfmrod/mapseq
- name: GNU make
  role: compiles and installs MAPseq via 'make' and 'make install' after ./configure
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:7d52557825b3357cecbcea46f559c31d5167614ba617ce9116b5c314eb58d9e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - MAPseq 1.2.6
  - autoconf
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 15
---

# rrna-reference-database-redundancy-reduction

## Summary

A benchmarking skill for systematically reducing the redundancy of a full-length rRNA reference database (removing near-identical sequences via clustering) and quantifying the resulting trade-off in read-classification accuracy, measured as F½ score, for MAPseq and competing rRNA classifiers.

## When to use

Use when designing or evaluating a reduced-redundancy rRNA reference database: you need to know how removing near-identical full-length sequences affects mapping/classification accuracy (F½ score) of MAPseq or competing tools.

## When NOT to use

- Input is already a MAPseq .mseq output or an OTU/taxa count table (mapseq -otucounts / -otutable stage) — this skill concerns reference database construction, not post-hoc summarization
- The benchmark reads lack ground-truth assignments, since F½ accuracy against known identities cannot be computed
- The goal is a pure speed benchmark rather than quantifying the accuracy cost of reference redundancy reduction

## Inputs

- Full-length rRNA reference database in nucleotide FASTA format with one or more tab-separated taxonomy assignment files (accession id + taxonomy, with a # header giving identity cutoffs and levels, e.g. '#cutoff: 0.00:0.08 0.70:0.35 ...')
- Known-identity benchmark read set: amplicon and shotgun rRNA reads with ground-truth taxonomic assignments (e.g. FASTQ/FASTA)
- Series of clustered/reduced reference FASTA files at progressively lower redundancy levels

## Outputs

- Reduced-redundancy reference database series (FASTA + taxonomy files per level)
- MAPseq .mseq classification outputs per redundancy level (best-guess taxonomy and OTU assignments with confidence values)
- Table of F½ classification accuracy versus reference redundancy level

## How to apply

Clone https://github.com/jfmrod/mapseq and build MAPseq v1.2.6 from source (./setup.sh to fetch eutils and reference data, ./bootstrap if cloned from git, then ./configure && make && make install, optionally ./configure --prefix=$HOME/usr for a home install). Load the full-length rRNA reference database plus a known-identity benchmark set of amplicon and shotgun rRNA reads with ground-truth taxonomic assignments. Generate a series of reduced reference databases by clustering and removing near-identical sequences at progressively lower redundancy levels. For each level, run MAPseq's k-mer search classification of the benchmark reads in a single MAPseq run, then compute the F½ (F1/2) classification accuracy against the ground truth and tabulate F½ versus redundancy level. The expected result — confirmed against Fig. 1g-h and Supplementary Fig. S2 — is that redundancy reduction carries a significant trade-off cost in mapping accuracy for all methods tested, so treat reference comprehensiveness as a non-negotiable accuracy parameter; use MAPseq's recommended combined confidence cutoff of 0.4 or 0.5, which yielded the highest F1/2-score in the authors' benchmarks.

## Related tools

- **MAPseq 1.2.6** (k-mer search classifier used to map known-identity rRNA reads against each redundancy-reduced reference and produce confidence-annotated taxonomy/OTU assignments) — https://github.com/jfmrod/mapseq
- **autoconf** (generates the 'configure' script used to build MAPseq from the cloned repository source) — https://github.com/jfmrod/mapseq
- **GNU make** (compiles and installs MAPseq via 'make' and 'make install' after ./configure)

## Examples

```
mapseq rawseqs.fa customref.fasta customref.tax > rawseqs.fa.mseq
```

## Evaluation signals

- F½ score is computed for every redundancy level against the ground-truth read assignments, producing a complete accuracy-versus-redundancy table
- The degradation trend in F½ with decreasing reference redundancy matches the behaviour reported in Fig. 1g-h and Supplementary Fig. S2
- Each reduced reference is verified to be a genuinely de-duplicated subset (near-identical sequences removed) at a strictly lower redundancy than the previous level
- All redundancy levels are classified within a single MAPseq run using the recommended combined confidence cutoff of 0.4 or 0.5
- MAPseq v1.2.6 builds and installs cleanly (./configure && make && make install completes without errors) before benchmarking begins

## Limitations

- Reducing reference redundancy has a significant trade-off cost in mapping accuracy for all methods tested, so a reduced database cannot be assumed accuracy-neutral
- The MAPseq binary must stay co-located with its data files: 'Moving only the binary elsewhere will break the installation though, as the data files are searched for in relation to the binary's path'
- Building from source requires additional dependencies (svn, autotools/autoconf, wget, git, libncurses5-dev, libtool); on macOS multi-arch builds may need the 'lipo' tool one architecture at a time

## Evidence

- [results] Reducing reference dataset redundancy by removing near-identical sequences has a significant trade-off cost in terms of mapping accuracy for all methods tested.: [evidence span withheld — anchor status: not_found]
- [abstract] MAPseq is up to 30% more accurate (F½ score) than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] Custom user-provided reference: mapseq accepts a nucleotide fasta reference plus one or more taxonomy files.: [evidence span withheld — anchor status: external_doc]
- [methods] Build-from-source steps: configure, make, make install.: "'cd' to the directory containing the package's source code and type './configure' to configure the package for your system."
- [readme] Recommended confidence cutoff of 0.4 or 0.5 yielded the highest F1/2-score in benchmarks.: [evidence span withheld — anchor status: external_doc]
