---
name: high-throughput-sequence-dereplication-format-checking
description: Use when preparing large dereplicated amplicon FASTA files (e.g. UniEuk
  SSU-rRNA V4, ~324M unique sequences) for swarm clustering and you must verify each
  entry carries an abundance annotation and correct linearized formatting.
when_to_use_negative:
- Input is already a feature/occurrence table or cluster output rather than dereplicated
  sequence FASTA
- Sequences lack abundance annotations and no dereplication step can be run — swarm
  will not work properly without occurrence counts
- Goal is global-threshold OTU picking (e.g. 97% similarity); swarm uses a local threshold
  d and converting a 97% threshold into a large d is an incorrect use
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: swarm v3
  role: Target clustering tool whose mandatory dereplication and FASTA abundance-annotation
    requirements this skill checks; strict dereplication of input sequences is now
    mandatory in swarm 3.0
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Used for pair-assembly of FASTQ/SFF files prior to dereplication and format
    checking
  repo: https://github.com/torognes/vsearch
- name: cutadapt
  role: Used to trim adaptors and primers before dereplication and format validation
  repo: https://code.google.com/p/cutadapt/
- name: frederic-mahe/swarm-tests
  role: Unit tests, including input format tests (test_input.sh), for checking swarm
    input/output behaviour
  repo: https://github.com/frederic-mahe/swarm-tests
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:7c1c19efab6d323dd07ba853df96d864b045c6d04d9698247c1eac3af3dd0745
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - cutadapt
  - frederic-mahe/swarm-tests
  - swarm v3
  - vsearch
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 5
---

# high-throughput-sequence-dereplication-format-checking

## Summary

Mandatory pre-clustering step for swarm v3: strictly dereplicate amplicon reads, merge identical sequences, sort by decreasing abundance, and verify the FASTA carries the required abundance annotations (e.g. seqID_1000) before clustering at tera-scale.

## When to use

Use when preparing large dereplicated amplicon FASTA files (e.g. UniEuk SSU-rRNA V4, ~324M unique sequences) for swarm clustering and you must verify each entry carries an abundance annotation and correct linearized formatting.

## When NOT to use

- Input is already a feature/occurrence table or cluster output rather than dereplicated sequence FASTA
- Sequences lack abundance annotations and no dereplication step can be run — swarm will not work properly without occurrence counts
- Goal is global-threshold OTU picking (e.g. 97% similarity); swarm uses a local threshold d and converting a 97% threshold into a large d is an incorrect use

## Inputs

- Raw or pair-assembled FASTQ/SFF amplicon reads (pair-assembled with vsearch, trimmed with cutadapt)
- Dereplicated FASTA file (amplicons.fasta) with unique sequence identifiers ending in abundance values, e.g. >seqID1_1000

## Outputs

- Validated dereplicated FASTA sorted by decreasing abundance, ready for swarm v3 clustering
- Format-check confirmation (unique IDs, present abundance annotations, linearized sequences)

## How to apply

Dereplication is mandatory in swarm v3: identify, merge, and sort identical sequences by decreasing abundance, since swarm requires each FASTA entry to present a number of occurrences (e.g. >seqID1_1000) to work properly; this abundance information can be produced with tools like vsearch during the dereplication step. Before clustering, format-check the input: sequence identifiers must be unique, abundances must be annotated (alternative formats possible with the -z option), and sequences should be linearized (--fasta_width 0) and possibly relabeled with hash values (--relabel_sha1). Confirm the memory footprint expectation of roughly 0.6 times the input FASTA size so the ~324M-sequence UniEuk V4 dataset can be planned for on the target machine. Only after these checks should clustering with swarm v3 (default d = 1) proceed, e.g. timed at 50 minutes on a 16-core system with --threads 16.

## Related tools

- **swarm v3** (Target clustering tool whose mandatory dereplication and FASTA abundance-annotation requirements this skill checks; strict dereplication of input sequences is now mandatory in swarm 3.0) — https://github.com/torognes/swarm
- **vsearch** (Used for pair-assembly of FASTQ/SFF files prior to dereplication and format checking) — https://github.com/torognes/vsearch
- **cutadapt** (Used to trim adaptors and primers before dereplication and format validation) — https://code.google.com/p/cutadapt/
- **frederic-mahe/swarm-tests** (Unit tests, including input format tests (test_input.sh), for checking swarm input/output behaviour) — https://github.com/frederic-mahe/swarm-tests

## Examples

```
vsearch --derep_fulllength amplicons.fastq --fasta_width 0 --relabel_sha1 --sizeout --output amplicons.fasta && ./swarm --threads 16 -o amplicons.swarms amplicons.fasta
```

## Evaluation signals

- Every FASTA entry has a unique identifier ending with an occurrence value (e.g. _1000) or an alternative abundance format accepted via -z
- Identical sequences are merged and entries sorted by decreasing abundance after dereplication
- Sequences are linearized (--fasta_width 0) with no wrapped lines, optionally relabeled with SHA1 hash values
- Memory footprint of the run is roughly 0.6 times the input FASTA size (larger if fastidious is used)
- Swarm output file (amplicons.swarms) exists in the space-separated seed-first format, confirming the input passed format checks

## Limitations

- Swarm requires each FASTA entry to present a number of occurrences; missing abundance annotations prevent proper clustering
- The fastidious option can significantly increase memory footprint; use -c and -y to cap memory consumption
- Dereplication is strict and mandatory in swarm 3.0, so non-dereplicated inputs are rejected by design
- Using d >= 2 instead of the default d = 1 caused a 15% slowdown in certain conditions (18SV9 dataset)

## Evidence

- [readme] strict dereplication of input sequences is now **mandatory**: "strict dereplication of input sequences is now **mandatory**"
- [methods] The role of the dereplication step is to identify, merge and sort identical sequences by decreasing abundance.: "The role of the dereplication step is to identify, merge and sort identical sequences by decreasing abundance."
- [readme] Swarm **requires** each fasta entry to present a number of occurrences to work properly.: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [methods] the linearization (`--fasta_width 0`): "the linearization (`--fasta_width 0`)"
- [readme] The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file.: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
