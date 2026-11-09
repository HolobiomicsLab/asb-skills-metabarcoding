---
name: adapter-fasta-file-generation
description: Use when single-end FASTQ data lacks a known adapter sequence and you
  need to auto-detect and export candidate adapter sequences (e.g. Illumina TruSeq)
  to a FASTA for trimming or inspection.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead, per
  the fastp README
- An adapter sequence is already known and supplied — fastp then trims directly against
  the provided sequence and automatic detection is unnecessary
- Data are paired-end, where fastp primarily relies on overlap analysis of PE reads
  rather than single-end k-mer seed detection
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: fastp
  role: Performs automatic adapter detection and trimming; its Algorithm 1 recovers
    adapter sequences that are written to the output FASTA
  repo: https://github.com/OpenGene/fastp
- name: C++
  role: Implementation language of fastp's adapter-detection and nucleotide-tree algorithms
- name: Python
  role: Used to read the first 1M FASTQ reads and compute 10-mer occurrence frequencies
    for seed selection
  repo: https://github.com/OpenGene/fastp/blob/master/parallel.py
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:6a61596c2e17663dc2993500fcb8821aa597a676eeab72b0cce7f5dbe7d86932
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# adapter-fasta-file-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates a FASTA file of adapter sequences automatically detected by fastp's Algorithm 1, which computes 10-mer seed frequencies from the first 1M FASTQ reads and extends seeds via nucleotide trees. It enables adapter trimming on single-end data without requiring adapters as input.

## When to use

Use when single-end FASTQ data lacks a known adapter sequence and you need to auto-detect and export candidate adapter sequences (e.g. Illumina TruSeq) to a FASTA for trimming or inspection.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead, per the fastp README
- An adapter sequence is already known and supplied — fastp then trims directly against the provided sequence and automatic detection is unnecessary
- Data are paired-end, where fastp primarily relies on overlap analysis of PE reads rather than single-end k-mer seed detection

## Inputs

- Single-end FASTQ file (optionally gzip-compressed) with adapter-containing reads
- k-mer length k=10
- Seed frequency threshold 0.0001
- Dominant-child threshold 0.9 for nucleotide-tree extension

## Outputs

- detected_adapters.fasta: FASTA file of detected adapter sequences, each >=6bp
- fastp HTML/JSON QC report containing detected adapter contents

## How to apply

Read the first 1,000,000 reads of the input FASTQ file and compute occurrence frequencies of all 10-mers (k=10); treat k-mers with frequency >0.0001 as candidate adapter seeds after removing low-complexity sequences. For each seed, build a nucleotide tree and extend it forward and backward by walking each node's dominant child, requiring >90% dominance at each branch, until the complete adapter is recovered. Discard recovered sequences shorter than 6bp, then write the remaining adapter sequence(s) to a FASTA file (detected_adapters.fasta). The rationale, per fastp, is that adapter-containing reads create overrepresented k-mers at the read tails, so frequency thresholds plus tree-based consensus extension recover the true adapter without user input.

## Related tools

- **fastp** (Performs automatic adapter detection and trimming; its Algorithm 1 recovers adapter sequences that are written to the output FASTA) — https://github.com/OpenGene/fastp
- **C++** (Implementation language of fastp's adapter-detection and nucleotide-tree algorithms)
- **Python** (Used to read the first 1M FASTQ reads and compute 10-mer occurrence frequencies for seed selection) — https://github.com/OpenGene/fastp/blob/master/parallel.py

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- detected_adapters.fasta parses as valid FASTA and contains at least one sequence of length >=6bp (sequences shorter than 6bp are skipped)
- For TruSeq-style data, the detected sequence corresponds to the known Illumina TruSeq adapter (e.g. AGATCGGAAGAGCACACGTCTGAACTCCAGTCA) reported in fastp documentation
- Seed k-mers (k=10) selected from the first 1M reads have occurrence frequency >0.0001 and pass the low-complexity filter
- Nucleotide-tree extension stops unless a child node has >90% dominance, so any recovered extension should trace a dominant path, not a low-support branch

## Limitations

- Detection relies on overrepresented tail k-mers, so data with very low adapter contamination may yield no seed above the 0.0001 frequency threshold
- Adapter sequences shorter than 6bp are not reported
- fastp is designed for short reads (Illumina NovaSeq, MGI); for long reads the fastplong tool should be used instead

## Evidence

- [methods] Adapter sequences can be automatically detected, which means you don't have to input the adapter sequences to trim them.: "cut adapters. Adapter sequences can be automatically detected, which means you don't have to input the adapter sequences to trim them."
- [abstract] It can perform quality control, adapter trimming, quality filtering, per-read quality pruning and many other operations with a single scan of the FASTQ data.: "It can perform quality control, adapter trimming, quality filtering, per-read quality pruning and many other operations with a single scan of the FASTQ data."
- [readme] This tool is designed for processing short reads (i.e. Illumina NovaSeq, MGI): "This tool is designed for processing short reads (i.e. Illumina NovaSeq, MGI), if you are looking for tools to process long reads (i.e. Nanopore, PacBio, Cyclone), please use [fastplong]"
