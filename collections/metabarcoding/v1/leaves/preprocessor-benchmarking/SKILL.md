---
name: preprocessor-benchmarking
description: Use when you need an empirical head-to-head comparison of FASTQ preprocessing
  tools, or must quantify how a preprocessor choice affects downstream variant-calling
  specificity (false-positive counts) on a dataset with a curated truth set.
when_to_use_negative:
- Reads are long reads (Nanopore, PacBio, Cyclone) — the fastp toolchain here targets
  short reads; use fastplong instead
- The goal is runtime/speed benchmarking alone — this skill measures downstream specificity
  via false-positive variant counts, not preprocessing speed
- No curated variant truth set exists for the sample, so false positives cannot be
  defined
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Preprocessor under test, run in paired-end mode with default adapter auto-detection
    and default quality filtering
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Comparison preprocessor with LEADING/TRAILING/SLIDINGWINDOW equivalents
- name: Cutadapt
  role: Comparison preprocessor with adapter-trimming settings comparable to fastp
    defaults
- name: SOAPnuke
  role: Comparison preprocessor with comparable adapter-trimming and quality-pruning
    settings
- name: SpeedSeq
  role: Held-constant aligner/variant caller applied to each preprocessed read pair
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:4c907844e352d59d4495418f8689c946954be41107ff7eed4e9c0548be9bee11
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
  - Cutadapt
  - SOAPnuke
  - SpeedSeq
  - Trimmomatic
  - fastp
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# preprocessor-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark FASTQ preprocessing tools by running each on the same raw paired-end reads and comparing downstream variant-calling false positives against a curated truth set. Grounded in the fastp paper's comparison of fastp, Trimmomatic, Cutadapt and SOAPnuke on NA12878 (SRR952827).

## When to use

Use when you need an empirical head-to-head comparison of FASTQ preprocessing tools, or must quantify how a preprocessor choice affects downstream variant-calling specificity (false-positive counts) on a dataset with a curated truth set.

## When NOT to use

- Reads are long reads (Nanopore, PacBio, Cyclone) — the fastp toolchain here targets short reads; use fastplong instead
- The goal is runtime/speed benchmarking alone — this skill measures downstream specificity via false-positive variant counts, not preprocessing speed
- No curated variant truth set exists for the sample, so false positives cannot be defined

## Inputs

- Paired-end FASTQ reads (gzipped R1/R2 pairs, e.g. NA12878 accession SRR952827)
- Curated variant truth set for the sample (NA12878 curated variants)
- Reference genome (FASTA) for alignment and variant calling
- Preprocessing tool binaries/configurations: fastp, Trimmomatic, Cutadapt, SOAPnuke
- Variant caller (SpeedSeq)

## Outputs

- Preprocessed paired-end FASTQ files per tool (out.R1.fq.gz / out.R2.fq.gz)
- Variant callset per preprocessor from SpeedSeq
- fp_counts.csv with false-positive variant counts per preprocessor (fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174)

## How to apply

Download the paired-end FASTQ reads (e.g. NA12878 accession SRR952827), the matching curated variant truth set, and the reference genome. Preprocess the same raw pairs with each tool under comparable settings: fastp in paired-end mode with default adapter auto-detection and default quality filtering (qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15); Trimmomatic, Cutadapt and SOAPnuke with equivalent adapter-trimming and sliding-window quality-pruning settings. Align each preprocessed pair and call variants with a single caller (SpeedSeq) so the caller is held constant. Intersect each callset with the curated variants and count false positives (called variants absent from the curated set). The benchmark is validated when the reproduced counts match the reference values: fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174, showing fastp yields the fewest false positives.

## Related tools

- **fastp** (Preprocessor under test, run in paired-end mode with default adapter auto-detection and default quality filtering) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Comparison preprocessor with LEADING/TRAILING/SLIDINGWINDOW equivalents)
- **Cutadapt** (Comparison preprocessor with adapter-trimming settings comparable to fastp defaults)
- **SOAPnuke** (Comparison preprocessor with comparable adapter-trimming and quality-pruning settings)
- **SpeedSeq** (Held-constant aligner/variant caller applied to each preprocessed read pair)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- fp_counts.csv reproduces the reported ordering: fastp 6708 < Cutadapt 6942 < SOAPnuke 7040 < Trimmomatic 7174 false positives
- Each tool is run on the identical raw SRR952827 R1/R2 FASTQ pair, differing only in preprocessor
- fastp filtering settings match the reported defaults: qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15
- The same SpeedSeq alignment/calling pipeline is applied to all four preprocessed pairs
- False positives are counted as called variants absent from the NA12878 curated variant set

## Limitations

- Counts depend on the specific caller (SpeedSeq) and truth set; swapping callers may change the ranking
- Comparison requires careful parity of settings across tools; fastp's adapter sequences are auto-detected while other tools need explicit configuration
- fastp's polyX tail trimming and low-complexity filter are disabled by default and would need explicit flags if included in the benchmark

## Evidence
