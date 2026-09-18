---
name: kmer-frequency-seed-selection
description: Use when trimming adapters from single-end Illumina-style FASTQ data
  and no adapter sequence is provided as input, or when you must reconstruct the unknown
  adapter contaminating reads (e.g. detecting TruSeq adapters like AGATCGGAAGAGCACACGTCTGAACTCCAGTCA).
when_to_use_negative:
- Adapter sequences are already known and provided (e.g. via fastp's --adapter_sequence);
  explicit trimming is more precise
- Data are long reads (Nanopore, PacBio, Cyclone) — the README directs such data to
  fastplong instead of fastp
- Paired-end data where overlap/base-correction analysis or PE-specific adapter detection
  is more appropriate than single-end k-mer seeding
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Implements the k-mer seed selection and nucleotide-tree adapter detection
    and then trims the detected adapters during a single FASTQ scan
  repo: https://github.com/OpenGene/fastp
- name: Cutadapt
  role: Comparator FASTQ preprocessing tool that requires adapter sequences as input,
    motivating fastp's automatic detection
- name: Trimmomatic
  role: Comparator FASTQ preprocessing tool that fastp outperforms 2-5x in speed while
    performing adapter detection automatically
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:60751b3b2f6321e5a2a1a9990b8a1362ef4d423d618ca74047e4d4a2d0fc58d4
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
  - Cutadapt
  - Python
  - Trimmomatic
  - fastp
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kmer-frequency-seed-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automatic adapter-sequence detection from single-end FASTQ data as implemented in fastp: high-frequency k-mers (k=10) sampled from the first 1,000,000 reads serve as adapter seeds, which are then extended via nucleotide trees with dominant-child walking (>90% dominance) until the full adapter sequence is recovered — no adapter sequences need to be supplied by the user.

## When to use

Use when trimming adapters from single-end Illumina-style FASTQ data and no adapter sequence is provided as input, or when you must reconstruct the unknown adapter contaminating reads (e.g. detecting TruSeq adapters like AGATCGGAAGAGCACACGTCTGAACTCCAGTCA).

## When NOT to use

- Adapter sequences are already known and provided (e.g. via fastp's --adapter_sequence); explicit trimming is more precise
- Data are long reads (Nanopore, PacBio, Cyclone) — the README directs such data to fastplong instead of fastp
- Paired-end data where overlap/base-correction analysis or PE-specific adapter detection is more appropriate than single-end k-mer seeding

## Inputs

- Single-end FASTQ file (optionally gzip-compressed), e.g. in.fq
- Sampling limit: first 1,000,000 reads
- k-mer length k=10 and seed frequency threshold 0.0001

## Outputs

- Detected adapter sequence(s) (each >=6bp), usable as trimming targets
- Cleaned FASTQ with adapters cut (e.g. out.fq)
- JSON and HTML quality-control reports including adapter-content statistics

## How to apply

Read the first 1,000,000 reads of the input FASTQ file and compute all 10-mers (k=10), counting their occurrence frequencies. Select k-mer seeds whose frequency exceeds 0.0001 as candidate adapter seeds, after removing low-complexity sequences. For each seed, build a nucleotide tree and extend the seed forward and backward by walking each node's dominant child, requiring >90% dominance at each branch, until the complete adapter sequence is recovered. Write recovered sequences (each >=6bp; shorter sequences are skipped) as detected adapters, then feed them to fastp's adapter-cutting step in the same single scan of the FASTQ data. Rationale: true adapter contamination creates a small set of overwhelmingly overrepresented 10-mers, so frequency-based seed selection plus majority-vote tree extension recovers the adapter without user input, keeping fastp 2-5x faster than Trimmomatic or Cutadapt.

## Related tools

- **fastp** (Implements the k-mer seed selection and nucleotide-tree adapter detection and then trims the detected adapters during a single FASTQ scan) — https://github.com/OpenGene/fastp
- **Cutadapt** (Comparator FASTQ preprocessing tool that requires adapter sequences as input, motivating fastp's automatic detection)
- **Trimmomatic** (Comparator FASTQ preprocessing tool that fastp outperforms 2-5x in speed while performing adapter detection automatically)

## Examples

```
fastp -i in.fq -o out.fq
```

## Evaluation signals

- Detected adapter sequences are >=6bp; sequences shorter than 6bp are skipped by the algorithm
- For TruSeq-style data the detected sequence should match the known Illumina TruSeq adapter (e.g. AGATCGGAAGAGCACACGTCTGAACTCCAGTCA) reported in the tool documentation
- Adapter content reported in the before/after QC profiling (JSON/HTML) drops substantially after trimming
- Seed k-mers were only accepted when occurrence frequency >0.0001 among the first 1,000,000 reads, after low-complexity removal
- Output FASTQ parses correctly and reads show adapter removal at the 3' ends

## Limitations

- Detection relies on overrepresented 10-mers in the first 1M reads; low-frequency or highly heterogeneous adapter contamination may not yield seeds exceeding the 0.0001 frequency threshold
- Low-complexity k-mers are removed from the seed pool, so polyG/polyX-style tails are handled by separate trimming features rather than by adapter detection
- The tool is designed for short reads (Illumina NovaSeq, MGI); for long reads the README redirects to fastplong

## Evidence
