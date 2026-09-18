---
name: nucleotide-tree-adapter-extension
description: Use when single-end FASTQ data needs adapter trimming but no adapter
  sequence is provided, and you must reproduce or re-implement fastp's seed-and-nucleotide-tree
  adapter detection (Algorithm 1).
when_to_use_negative:
- Adapter sequences are already known and supplied on the command line, so automatic
  detection adds no value
- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead of fastp
- Detection relies on read frequency statistics, so very small input files (< the
  sampling depth at which adapter k-mers reach frequency >0.0001) may yield no reliable
  seeds
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Implements the automatic adapter detection and trimming; the algorithm being
    reconstructed is fastp's Algorithm 1
  repo: https://github.com/OpenGene/fastp
- name: Python
  role: Used to read the first 1M FASTQ reads and count 10-mer occurrence frequencies
    when re-implementing the algorithm
  repo: https://github.com/OpenGene/fastp/blob/master/parallel.py
- name: C++
  role: Implementation language of fastp and the nucleotide-tree adapter detection
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:1739c21cff514a5414c48e6d89e61bb9baf72b3ebe4cb4d172fbbf3be25ec26b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/OpenGene/fastp
  repo_url: https://github.com/OpenGene/fastp
  tools:
  - C++
  - Python
  - fastp
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# nucleotide-tree-adapter-extension

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs fastp's automatic adapter-detection algorithm: high-frequency k-mer seeds from the first 1M FASTQ reads are extended forward and backward using nucleotide trees with a >90% dominant-child rule to recover full adapter sequences without user-supplied adapter input.

## When to use

Use when single-end FASTQ data needs adapter trimming but no adapter sequence is provided, and you must reproduce or re-implement fastp's seed-and-nucleotide-tree adapter detection (Algorithm 1).

## When NOT to use

- Adapter sequences are already known and supplied on the command line, so automatic detection adds no value
- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead of fastp
- Detection relies on read frequency statistics, so very small input files (< the sampling depth at which adapter k-mers reach frequency >0.0001) may yield no reliable seeds

## Inputs

- Single-end FASTQ file (e.g. Illumina NovaSeq/MGI short reads)
- k-mer length k=10
- Sample limit of 1,000,000 reads
- Seed frequency threshold 0.0001
- Dominant-child dominance threshold 90%

## Outputs

- detected_adapters.fasta containing recovered adapter sequences (each >=6bp)
- Adapter sequences used downstream by fastp's adapter trimming step

## How to apply

Read the first 1,000,000 reads of the input FASTQ file and compute all 10-mers (k=10) across the read sequences, counting occurrence frequencies. Discard low-complexity sequences, then keep k-mer seeds whose occurrence frequency exceeds 0.0001 as candidate adapter seeds. For each seed, build a nucleotide tree and extend the seed forward and backward by walking each node's dominant child, requiring >90% dominance at each branch, until the complete adapter sequence is recovered. Sequences shorter than 6bp are skipped; write the recovered adapter(s) to a FASTA file (e.g. detected_adapters.fasta) and feed them to adapter trimming.

## Related tools

- **fastp** (Implements the automatic adapter detection and trimming; the algorithm being reconstructed is fastp's Algorithm 1) — https://github.com/OpenGene/fastp
- **Python** (Used to read the first 1M FASTQ reads and count 10-mer occurrence frequencies when re-implementing the algorithm) — https://github.com/OpenGene/fastp/blob/master/parallel.py
- **C++** (Implementation language of fastp and the nucleotide-tree adapter detection) — https://github.com/OpenGene/fastp

## Examples

```
fastp -i in.fq -o out.fq --detect_adapter_for_pe
```

## Evaluation signals

- detected_adapters.fasta parses as valid FASTA and contains at least one sequence of at least 6bp
- On TruSeq-style data the detected sequence corresponds to the known Illumina TruSeq adapter, e.g. AGATCGGAAGAGCACACGTCTGAACTCCAGTCA
- All retained seeds have 10-mer occurrence frequency >0.0001 after low-complexity removal
- Every extension step walked a child node with >90% dominance; extensions that fail the threshold terminate
- No recovered adapter shorter than 6bp appears in the output

## Limitations

- Detection depends on adapters appearing frequently enough in the first 1M reads; low adapter-content libraries may yield no seed passing the 0.0001 frequency threshold
- Low-complexity sequences are removed before seeding, so low-complexity adapter-like tails are not detected by this route
- fastp is designed for short reads (Illumina NovaSeq, MGI); long reads require fastplong, and the README notes adapter trimming is one feature among quality filtering, polyG/polyX trimming, UMI processing etc.

## Evidence
