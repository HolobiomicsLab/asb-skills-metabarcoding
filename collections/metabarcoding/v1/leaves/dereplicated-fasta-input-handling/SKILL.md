---
name: dereplicated-fasta-input-handling
description: Use when preparing amplicon FASTA files for swarm clustering, or when
  benchmarking/running swarm (e.g., v2 vs v3) on large dereplicated SSU-rRNA datasets
  such as 10.6M-sequence V4 or V9 files.
when_to_use_negative:
- Input is a filtered occurrence table or an OTU/abundance table rather than raw or
  dereplicated sequence FASTA
- Sequences have not been pair-assembled, trimmed, or dereplicated — swarm requires
  occurrence counts per entry and clustering on non-dereplicated input is incorrect
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3293
tools:
- name: swarm
  role: Clusters the dereplicated FASTA into OTUs; validates input by requiring occurrence
    counts per entry
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Pair-assembles SFF or FASTQ reads prior to dereplication and clustering
  repo: https://github.com/torognes/vsearch
- name: cutadapt
  role: Trims adaptors and primers from reads before dereplication
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:2e7147aac5b867a0619d1cb0b5eba7ff4182ceec154602a448724005d2a10abc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - cutadapt
  - swarm
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# dereplicated-fasta-input-handling

## Summary

Prepares and validates dereplicated amplicon FASTA files so swarm can cluster them correctly: entries must be linearized, uniquely labeled, and carry occurrence counts, since strict dereplication is mandatory in swarm 3 and clustering quality depends on this format.

## When to use

Use when preparing amplicon FASTA files for swarm clustering, or when benchmarking/running swarm (e.g., v2 vs v3) on large dereplicated SSU-rRNA datasets such as 10.6M-sequence V4 or V9 files.

## When NOT to use

- Input is a filtered occurrence table or an OTU/abundance table rather than raw or dereplicated sequence FASTA
- Sequences have not been pair-assembled, trimmed, or dereplicated — swarm requires occurrence counts per entry and clustering on non-dereplicated input is incorrect

## Inputs

- Dereplicated amplicon FASTA file with unique sequence identifiers ending in occurrence counts (e.g., >seqID2_25)
- Raw SFF or FASTQ read files (pre pair-assembly and trimming)
- Alternative occurrence-count format accepted via the -z option

## Outputs

- Linearized, hash-relabelable dereplicated FASTA suitable as swarm input
- Swarm clusters / OTU seeds written via --seeds (-w), sorted by decreasing abundance then alphabetical label order

## How to apply

First ensure reads are pair-assembled (e.g., with vsearch) and trimmed of adaptors and primers (e.g., with cutadapt). Then dereplicate: identify, merge, and sort identical sequences by decreasing abundance, because swarm requires each FASTA entry to end with an occurrence count (e.g., >seqID1_1000; alternative format via -z). Linearize the FASTA (--fasta_width 0) and optionally relabel with hash values (--relabel_sha1) before clustering. Note that swarm 3 makes strict dereplication mandatory, and its memory footprint is roughly 0.6x the input FASTA size, so dataset scale should be anticipated. For benchmark-style reuse, write cluster representatives with --seeds/-w and confirm the input files are identical across software versions to guarantee comparable results.

## Related tools

- **swarm** (Clusters the dereplicated FASTA into OTUs; validates input by requiring occurrence counts per entry) — https://github.com/torognes/swarm
- **vsearch** (Pair-assembles SFF or FASTQ reads prior to dereplication and clustering) — https://github.com/torognes/vsearch
- **cutadapt** (Trims adaptors and primers from reads before dereplication)

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- Every FASTA header ends with an abundance value (or -z alternative format) and identifiers are unique
- Sequences are sorted by decreasing abundance after dereplication
- swarm runs without input-format errors; memory usage stays near 0.6x the input FASTA size
- Re-running on the same dereplicated input yields identical clusters (swarm results are resilient to input-order changes)
- In benchmark settings, the v3 --seeds output matches the v2 reference output exactly (byte-identical OTUs)

## Limitations

- Strict dereplication is mandatory in swarm 3; files lacking occurrence counts cannot be processed correctly
- Memory footprint scales with input size (~0.6x input FASTA, more with --fastidious; control with -c and -y)
- FASTA must be linearized (--fasta_width 0) and properly labeled; misformatted headers silently break abundance parsing

## Evidence

- [methods] The role of the dereplication step is to identify, merge and sort identical sequences by decreasing abundance.: "The role of the dereplication step is to identify, merge and sort identical sequences by decreasing abundance."
- [readme] strict dereplication mandatory: "strict dereplication of input sequences is now **mandatory**"
- [readme] fasta entry occurrence count: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [readme] memory footprint 0.6x input: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
