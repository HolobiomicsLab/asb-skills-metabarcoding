---
name: fastq-adapter-sequence-auto-detection
description: Use when trimming Illumina short-read FASTQ data without knowing the
  adapter sequence, and you need to recover or reimplement fastp's seed-and-nucleotide-tree
  adapter detection.
when_to_use_negative:
- Processing long reads (Nanopore, PacBio, Cyclone) — the README directs those inputs
  to fastplong instead of fastp
- Adapter sequences are already known and provided as input — fastp's auto-detection
  is specifically for when you don't have to input the adapter sequences
- Data quality/composition makes high-frequency 10-mers reflect genomic content rather
  than adapter (e.g. low-diversity libraries with low-complexity seeds not filtered
  out)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3173
tools:
- name: fastp
  role: Implements automatic adapter detection and adapter trimming during FASTQ preprocessing;
    the algorithm being reconstructed
  repo: https://github.com/OpenGene/fastp
- name: Python
  role: Used to read the first 1,000,000 FASTQ reads, compute 10-mer frequencies,
    and select seed k-mers with frequency >0.0001
- name: C++
  role: Implementation language of fastp with multi-threading support, in which the
    nucleotide-tree extension logic operates
  repo: https://github.com/OpenGene/fastp
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:73fe77d28da620cf191a897229e69f7470ecd627dba785f7803b17f44338618f
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
  - http://edamontology.org/topic_3173
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# fastq-adapter-sequence-auto-detection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs fastp's automatic adapter-detection algorithm (Algorithm 1): adapter seeds are found as high-frequency 10-mers in the first 1M reads of a single-end FASTQ file, then extended via nucleotide-tree traversal into complete adapter sequences without requiring adapter input.

## When to use

Use when trimming Illumina short-read FASTQ data without knowing the adapter sequence, and you need to recover or reimplement fastp's seed-and-nucleotide-tree adapter detection.

## When NOT to use

- Processing long reads (Nanopore, PacBio, Cyclone) — the README directs those inputs to fastplong instead of fastp
- Adapter sequences are already known and provided as input — fastp's auto-detection is specifically for when you don't have to input the adapter sequences
- Data quality/composition makes high-frequency 10-mers reflect genomic content rather than adapter (e.g. low-diversity libraries with low-complexity seeds not filtered out)

## Inputs

- Single-end FASTQ file (Illumina short reads, e.g. NovaSeq/MGI data)
- k-mer length parameter k=10
- seed frequency threshold 0.0001
- dominant-child threshold 90%
- minimum adapter length 6bp

## Outputs

- detected_adapters.fasta containing recovered adapter sequences (each >=6bp)
- Validated FASTA where TruSeq-style data yields the known Illumina TruSeq adapter (e.g. AGATCGGAAGAGCACACGTCTGAACTCCAGTCA)

## How to apply

Read the first 1,000,000 reads of the input FASTQ file and count all k-mers of length k=10. Treat k-mers whose occurrence frequency exceeds 0.0001 as candidate adapter seeds, discarding low-complexity sequences to avoid false seeds. For each seed, build a nucleotide tree over the read corpus and extend the seed forward and backward by walking each node's dominant child (>90% dominance percentage) until the complete adapter sequence is recovered. Sequences shorter than 6bp are skipped; write recovered adapters to a FASTA file. On TruSeq-style data the recovered sequence should match the known Illumina TruSeq adapter, e.g. AGATCGGAAGAGCACACGTCTGAACTCCAGTCA.

## Related tools

- **fastp** (Implements automatic adapter detection and adapter trimming during FASTQ preprocessing; the algorithm being reconstructed) — https://github.com/OpenGene/fastp
- **Python** (Used to read the first 1,000,000 FASTQ reads, compute 10-mer frequencies, and select seed k-mers with frequency >0.0001)
- **C++** (Implementation language of fastp with multi-threading support, in which the nucleotide-tree extension logic operates) — https://github.com/OpenGene/fastp

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- Output FASTA (detected_adapters.fasta) parses correctly as a FASTA file
- At least one detected adapter sequence has length >= 6bp (shorter sequences are skipped by the algorithm)
- For TruSeq-style single-end data, the detected sequence matches the known Illumina TruSeq adapter AGATCGGAAGAGCACACGTCTGAACTCCAGTCA
- Candidate seeds all have 10-mer occurrence frequency > 0.0001 and low-complexity k-mers are removed before extension
- Nucleotide-tree extension only follows child nodes with >90% dominant percentage, stopping when dominance is lost

## Limitations

- Sequences shorter than 6bp are skipped, so very short adapter remnants are not recovered
- Low-complexity sequences must be removed before seed selection to avoid spurious high-frequency k-mers
- Detection relies on the first 1,000,000 reads; adapter content in very shallow or contaminated datasets may not produce seeds above the 0.0001 frequency threshold
- The 90% dominant-child requirement may terminate extension early when read quality or adapter coverage is low

## Evidence
