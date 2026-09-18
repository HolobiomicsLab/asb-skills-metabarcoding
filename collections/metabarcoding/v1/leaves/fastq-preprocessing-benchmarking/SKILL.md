---
name: fastq-preprocessing-benchmarking
description: Use when the goal is to quantify how fast and how well FASTQ preprocessing
  tools (fastp vs Trimmomatic, Cutadapt, AfterQC, SOAPnuke, FASTQC, umis, UMI-tools)
  perform on Illumina paired-end FASTQ data.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — the paper's fastp targets
  short reads (Illumina NovaSeq, MGI); use fastplong instead
- Input data is already adapter-trimmed and quality-filtered, so no preprocessing
  comparison is meaningful
- Comparison requires multi-threaded scaling results — this skill benchmarks single-threaded
  performance (-w 1) only, and timings are hardware-dependent
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: fastp
  role: Ultra-fast all-in-one FASTQ preprocessor under benchmark; performs quality
    control, adapter trimming, quality filtering and per-read quality pruning in a
    single scan
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Java-based comparison tool run single-threaded with ILLUMINACLIP plus sliding-window
    quality pruning; benchmarks 3-5x slower than fastp
- name: Cutadapt
  role: Adapter-removal comparison tool supplied the 33 bp adapter sequence explicitly
- name: FASTQC
  role: Second-fastest comparison tool; reference point for throughput ranking
- name: AfterQC
  role: Comparison tool run in default adapter-trimming mode in speed and residue-rate
    benchmarks
- name: SOAPnuke
  role: Comparison tool run in default adapter-trimming mode in speed and residue-rate
    benchmarks
- name: umis
  role: Python UMI-extraction tool compared in the UMI preprocessing benchmark (12.4
    min on 4 Gb PE150)
- name: UMI-tools
  role: Slowest UMI extraction tool in the benchmark (28.3 min on 4 Gb PE150)
- name: Python
  role: Scripting language used to scan filtered reads for the 33 bp adapter with
    a mismatch sweep and to compute throughput
  repo: https://github.com/OpenGene/fastp/blob/master/parallel.py
- name: Java
  role: Runtime for the single-threaded Trimmomatic benchmark
merged_aliases:
- fastq-adapter-trimming-benchmarking
merged_alias_records:
- alias: fastq-adapter-trimming-benchmarking
  slug: fastq-adapter-trimming-benchmarking
  cosine_score: 0.8297
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_005
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:e5c45b91c7d77520164aaccf838691bdbfc2d3248b22280d7e6273f313e3f0f8
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
  - AfterQC
  - Cutadapt
  - FASTQC
  - Java
  - Python
  - SOAPnuke
  - Trimmomatic
  - UMI-tools
  - fastp
  - umis
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# fastq-preprocessing-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark fastp against alternative FASTQ preprocessors (FASTQC, Cutadapt, SOAPnuke, AfterQC, Trimmomatic, umis, UMI-tools) on the same FASTQ input, comparing single-threaded wall-clock processing time/throughput and residual adapter content to reproduce the fastp paper's Tables 1, 3 and Figure 6.

## When to use

Use when the goal is to quantify how fast and how well FASTQ preprocessing tools (fastp vs Trimmomatic, Cutadapt, AfterQC, SOAPnuke, FASTQC, umis, UMI-tools) perform on Illumina paired-end FASTQ data.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — the paper's fastp targets short reads (Illumina NovaSeq, MGI); use fastplong instead
- Input data is already adapter-trimmed and quality-filtered, so no preprocessing comparison is meaningful
- Comparison requires multi-threaded scaling results — this skill benchmarks single-threaded performance (-w 1) only, and timings are hardware-dependent

## Inputs

- Paired-end (R1/R2, gzip-compressed) Illumina FASTQ files, e.g. the B17NCB1 dataset (9316 M bases) or a 4 Gb PE150 dataset with UMI at the head of read1
- Single-end (uncompressed) FASTQ file for SE-mode benchmarks
- Known adapter sequence (33 bp) for residual-adapter scanning
- UMI length and UMI location (read1) specification

## Outputs

- Benchmark table (e.g. Table 1-style CSV, benchmark_results.csv) with processing time and reads-per-second for each tool in SE and PE modes
- UMI preprocessing benchmark table reproducing Table 3 (fastp 4.6 min, umis 12.4 min, UMI-tools 28.3 min)
- Adapter residue-rate table and Figure 6-style plot of suspected-adapter rate vs allowed mismatches (0-5) per tool
- fastp quality-control reports (fastp.html, fastp.) and trimmed FASTQ outputs per tool

## How to apply

Run all tools single-threaded on the identical dataset (e.g. the paired-end B17NCB1 FASTQ, 9316 M bases) in both SE and PE modes, recording wall-clock time: fastp is invoked with -w 1 as `fastp -i in.fq -o out.fq` (SE) or `fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz` (PE), capturing fastp.html and fastp. reports, while Trimmomatic runs via Java with ILLUMINACLIP plus sliding-window quality pruning and Cutadapt is given the adapter sequence explicitly. Compute throughput as total reads processed divided by time (fastp PE ≈ 116750 reads/s at 13.3 min vs Trimmomatic PE 60.9 min) and assemble a Table 1-style CSV of all tool-by-mode rows; expect fastp ~2x faster than the second-fastest FASTQC and 3-5x faster than the rest, consistent with the reported 2-5x speedup. For UMI preprocessing on 4 Gb PE150 data, run fastp with `-U --umi_loc=read1 --umi_len=<len>`, umis extract, and UMI-tools extract, expecting the ordering fastp 4.6 min < umis 12.4 min < UMI-tools 28.3 min (~2.7x and ~6.1x speedups). For adapter-removal quality, scan each tool's filtered output with a Python script for the 33 bp adapter sequence while sweeping allowed mismatches (0-5); fastp with --detect_adapter_for_pe and Trimmomatic should show the lowest residue rates, with zero suspected adapters at ≤4 mismatches.

## Related tools

- **fastp** (Ultra-fast all-in-one FASTQ preprocessor under benchmark; performs quality control, adapter trimming, quality filtering and per-read quality pruning in a single scan) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Java-based comparison tool run single-threaded with ILLUMINACLIP plus sliding-window quality pruning; benchmarks 3-5x slower than fastp)
- **Cutadapt** (Adapter-removal comparison tool supplied the 33 bp adapter sequence explicitly)
- **FASTQC** (Second-fastest comparison tool; reference point for throughput ranking)
- **AfterQC** (Comparison tool run in default adapter-trimming mode in speed and residue-rate benchmarks)
- **SOAPnuke** (Comparison tool run in default adapter-trimming mode in speed and residue-rate benchmarks)
- **umis** (Python UMI-extraction tool compared in the UMI preprocessing benchmark (12.4 min on 4 Gb PE150))
- **UMI-tools** (Slowest UMI extraction tool in the benchmark (28.3 min on 4 Gb PE150))
- **Python** (Scripting language used to scan filtered reads for the 33 bp adapter with a mismatch sweep and to compute throughput) — https://github.com/OpenGene/fastp/blob/master/parallel.py
- **Java** (Runtime for the single-threaded Trimmomatic benchmark)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -w 1 --detect_adapter_for_pe -- fastp. --html fastp.html
```

## Evaluation signals

- The benchmark table contains all 12 tool-by-mode rows (6 tools x SE/PE) and fastp PE/SE timings reproduce the reported 2-5x speedup over Trimmomatic and Cutadapt
- Throughput values are internally consistent (reads processed / wall-clock time), e.g. fastp PE 13.3 min ≈ 116750 reads/s
- UMI benchmark reproduces Table 3 ordering and magnitudes: fastp ~4.6 min < umis ~12.4 min < UMI-tools ~28.3 min, and each tool emits a UMI-shifted FASTQ without error
- Residual-adapter scan shows fastp-filtered data contains zero suspected 33 bp adapters when ≤4 mismatches are allowed, with fastp and Trimmomatic ranked lowest overall
- fastp runs complete without error and emit valid fastp.html and fastp. reports

## Limitations

- All timings are single-threaded (-w 1) wall-clock measurements on one hardware setup; absolute times are not portable across machines
- fastp's speed advantage comes partly from being written in C++ with multi-threading support, while comparison tools are Python/Java-based — the comparison conflates implementation language with algorithmic efficiency
- fastp is designed for short reads only; long-read (Nanopore, PacBio) preprocessing requires fastplong, and polyX tail trimming is disabled by default (needs -x)

## Evidence
