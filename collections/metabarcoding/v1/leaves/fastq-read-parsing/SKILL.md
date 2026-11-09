---
name: fastq-read-parsing
description: Use when single-end FASTQ data must be parsed to detect adapter sequences
  automatically (no adapter given as input), or when reproducing fastp's k-mer seed
  plus nucleotide-tree adapter-detection algorithm from raw read files.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp/fastq-read-parsing targets
  short reads; use fastplong instead
- Adapter sequences are already known and supplied explicitly — automatic detection
  via k-mer seeds is unnecessary overhead
- Data are not FASTQ (e.g. SAM/BAM or feature tables) — the parser assumes FASTQ record
  structure
edam_operation: http://edamontology.org/operation_3628
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: 'Primary tool: ultra-fast C++ FASTQ preprocessor that automatically detects
    and trims adapter sequences in a single scan of the FASTQ data'
  repo: https://github.com/OpenGene/fastp
- name: Python
  role: Scripting layer used to read the first 1,000,000 FASTQ reads, count 10-mer
    frequencies, and write detected_adapters.fasta when re-implementing the algorithm
- name: C++
  role: Implementation language of fastp with multi-threading support, enabling the
    single-scan parsing and nucleotide-tree extension
  repo: https://github.com/OpenGene/fastp
- name: fastplong
  role: Alternative tool for long reads where this short-read parsing skill does not
    apply
  repo: https://github.com/OpenGene/fastplong
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:8b19deaf3ebcab031bff82a08cc30d7f6fb6fcaf21e653198e6e8c9390729676
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
  - fastplong
  edam_operation: http://edamontology.org/operation_3628
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# fastq-read-parsing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Parse and scan FASTQ short-read data in a single pass to support fastp-style automatic adapter detection, counting k-mer seeds (k=10) over the first 1,000,000 reads and extending dominant-branch nucleotide trees to recover full adapter sequences without user-supplied adapter input.

## When to use

Use when single-end FASTQ data must be parsed to detect adapter sequences automatically (no adapter given as input), or when reproducing fastp's k-mer seed plus nucleotide-tree adapter-detection algorithm from raw read files.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — fastp/fastq-read-parsing targets short reads; use fastplong instead
- Adapter sequences are already known and supplied explicitly — automatic detection via k-mer seeds is unnecessary overhead
- Data are not FASTQ (e.g. SAM/BAM or feature tables) — the parser assumes FASTQ record structure

## Inputs

- Single-end FASTQ file (raw, optionally gzip-compressed) short-read data, e.g. Illumina NovaSeq/MGI reads
- k-mer length parameter k=10
- Seed frequency threshold 0.0001
- Dominant-child percentage threshold 90%

## Outputs

- detected_adapters.fasta containing recovered adapter sequences, each >=6bp
- Adapter-trimmed FASTQ output and fastp JSON/HTML QC report showing adapter content before/after

## How to apply

Read the first 1,000,000 records of the input FASTQ file and compute all 10-mers across the read sequences, tallying occurrence frequencies. Treat 10-mers with frequency >0.0001 as candidate adapter seeds after discarding low-complexity sequences. For each surviving seed, build a nucleotide tree and extend the seed forward and backward by walking each node's dominant child, requiring >90% dominance at every branch, until the complete adapter sequence is recovered. Emit recovered sequences (each >=6bp; shorter ones are skipped) to detected_adapters.fasta, and validate against a known adapter such as the Illumina TruSeq sequence AGATCGGAAGAGCACACGTCTGAACTCCAGTCA for TruSeq-style data. Rationale: fastp performs these operations in a single scan of the FASTQ data, which is what makes it 2-5x faster than Trimmomatic or Cutadapt while still detecting adapters without explicit input.

## Related tools

- **fastp** (Primary tool: ultra-fast C++ FASTQ preprocessor that automatically detects and trims adapter sequences in a single scan of the FASTQ data) — https://github.com/OpenGene/fastp
- **Python** (Scripting layer used to read the first 1,000,000 FASTQ reads, count 10-mer frequencies, and write detected_adapters.fasta when re-implementing the algorithm)
- **C++** (Implementation language of fastp with multi-threading support, enabling the single-scan parsing and nucleotide-tree extension) — https://github.com/OpenGene/fastp
- **fastplong** (Alternative tool for long reads where this short-read parsing skill does not apply) — https://github.com/OpenGene/fastplong

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- detected_adapters.fasta parses correctly as FASTA and contains at least one adapter sequence of at least 6bp (shorter candidates are deliberately skipped)
- For TruSeq-style single-end data, the detected sequence matches the documented Illumina TruSeq adapter AGATCGGAAGAGCACACGTCTGAACTCCAGTCA
- Seed candidates respect the occurrence-frequency cutoff (>0.0001) and low-complexity 10-mers are excluded before tree extension
- Every branch walked in the nucleotide tree satisfies the >90% dominant-child percentage; extension stops otherwise
- The whole pipeline completes within one scan of the input FASTQ (fastp is 2-5x faster than Trimmomatic/Cutadapt), consistent with the tool's single-pass design

## Limitations

- Adapter sequences shorter than 6bp are skipped during output, so very short adapter remnants may go undetected
- Automatic adapter detection depends on seed frequency >0.0001 among the first 1M reads; low-abundance or highly heterogeneous adapters may fail to seed a tree
- Designed for short reads (Illumina NovaSeq, MGI); long-read data (Nanopore, PacBio, Cyclone) require fastplong instead

## Evidence
