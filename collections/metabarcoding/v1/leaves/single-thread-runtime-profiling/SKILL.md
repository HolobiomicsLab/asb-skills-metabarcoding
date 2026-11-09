---
name: single-thread-runtime-profiling
description: Use when you need to compare the runtime or throughput of FASTQ preprocessors
  under a fixed, fair setting (single thread, same dataset such as the 9316 M-base
  B17NCB1 paired-end data) and reproduce the 2-5x speedup claims for fastp.
when_to_use_negative:
- The goal is to measure multi-threaded or I/O-bound performance — the skill specifically
  pins runs to a single thread (-w 1) to isolate algorithmic speed.
- Input data are long reads (Nanopore, PacBio, Cyclone) — the fastp README directs
  those workflows to fastplong instead.
- You need functional quality metrics (adapter removal rate, Q30 improvement) rather
  than runtime/throughput comparison — this skill only profiles speed.
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: fastp
  role: Target tool being benchmarked; run single-threaded (-w 1) in SE and PE modes
    and timed
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Comparison tool; second-fastest in the benchmark at about twice fastp's runtime
- name: Cutadapt
  role: Comparison tool; one of the tools fastp is reported to be 2-5 times faster
    than
- name: SOAPnuke
  role: Comparison tool in the six-tool SE/PE runtime table
- name: AfterQC
  role: Comparison tool; PE runtime 57.2 min vs fastp's 13.3 min
- name: Trimmomatic
  role: Java-based comparison tool; slowest PE runtime at 60.9 min
- name: Java
  role: Runtime for Trimmomatic, pinned to a single thread for fair comparison
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:9fb70decee22b16644eba3ceeb8349168857cbc3b60e5dff2329ed4bc37ac4a5
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
  - SOAPnuke
  - Trimmomatic
  - fastp
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-thread-runtime-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark methodology for measuring wall-clock processing time and throughput of FASTQ preprocessing tools (fastp vs FASTQC, Cutadapt, SOAPnuke, AfterQC, Trimmomatic) on a fixed dataset in single-threaded SE and PE modes.

## When to use

Use when you need to compare the runtime or throughput of FASTQ preprocessors under a fixed, fair setting (single thread, same dataset such as the 9316 M-base B17NCB1 paired-end data) and reproduce the 2-5x speedup claims for fastp.

## When NOT to use

- The goal is to measure multi-threaded or I/O-bound performance — the skill specifically pins runs to a single thread (-w 1) to isolate algorithmic speed.
- Input data are long reads (Nanopore, PacBio, Cyclone) — the fastp README directs those workflows to fastplong instead.
- You need functional quality metrics (adapter removal rate, Q30 improvement) rather than runtime/throughput comparison — this skill only profiles speed.

## Inputs

- Paired-end FASTQ files (e.g. B17NCB1 R1/R2, .fq.gz, 9316 M bases total)
- Single-end FASTQ file for SE-mode runs (in.fq)
- Installed binaries of fastp, FASTQC, Cutadapt, SOAPnuke, AfterQC, and Trimmomatic (Java, single-thread)

## Outputs

- Wall-clock processing times per tool per mode (SE and PE)
- Throughput table in reads per second per tool per mode
- Table 1-style CSV with 12 tool-by-mode rows (6 tools × 2 modes)
- fastp HTML/JSON quality-control reports from each fastp run

## How to apply

Obtain one fixed paired-end FASTQ dataset (e.g. B17NCB1, 9316 M bases) and hold hardware, threads (-w 1 for fastp), and input files constant across all tools. Run each of the six tools (fastp, FASTQC, Cutadapt, SOAPnuke, AfterQC, Trimmomatic via Java) in both SE and PE modes on identical inputs, timing every run to compute reads-per-second throughput (e.g. fastp PE 13.3 min ≈ 116750 reads/s vs AfterQC PE 57.2 min and Trimmomatic PE 60.9 min). Capture fastp's HTML/JSON reports alongside timing so QC output completeness can be checked, not just speed. Assemble all 12 tool-by-mode rows (6 tools × SE/PE) into a Table 1-style CSV with both time and throughput. The rationale, per the article, is that fastp is written in C++ with multi-threading support and performs quality control, adapter trimming, and quality filtering in a single scan, so single-thread timing isolates the core implementation speed advantage (2-5x over Trimmomatic/Cutadapt).

## Related tools

- **fastp** (Target tool being benchmarked; run single-threaded (-w 1) in SE and PE modes and timed) — https://github.com/OpenGene/fastp
- **FASTQC** (Comparison tool; second-fastest in the benchmark at about twice fastp's runtime)
- **Cutadapt** (Comparison tool; one of the tools fastp is reported to be 2-5 times faster than)
- **SOAPnuke** (Comparison tool in the six-tool SE/PE runtime table)
- **AfterQC** (Comparison tool; PE runtime 57.2 min vs fastp's 13.3 min)
- **Trimmomatic** (Java-based comparison tool; slowest PE runtime at 60.9 min)
- **Java** (Runtime for Trimmomatic, pinned to a single thread for fair comparison)

## Examples

```
fastp -w 1 -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- The output CSV contains exactly 12 rows (6 tools × SE/PE modes) with both processing time and reads-per-second throughput columns.
- The fastp PE timing is about 2-5x faster than Trimmomatic and Cutadapt (e.g. 13.3 min fastp PE vs 57.2 min AfterQC and 60.9 min Trimmomatic PE).
- All runs used identical input files and single-thread settings, so timing differences are attributable to tool implementation, not hardware or parallelism.
- Throughput values are internally consistent: reads/s = total reads processed / wall-clock time for each run.
- fastp runs also produced fastp.html and JSON reports, confirming preprocessing actually completed rather than timing a failed run.

## Limitations

- The 2-5x speedup claim is specific to single-threaded runs on the tested dataset (B17NCB1); multi-threaded fastp can widen or change the gap since it supports multi-threading.
- fastp is designed for short reads (Illumina NovaSeq, MGI); long-read data require fastplong, so this benchmark does not transfer to long-read preprocessing.
- The benchmark compares wall-clock speed; tools may perform different amounts of work, though the article notes fastp is faster 'despite performing far more operations than similar tools'.

## Evidence
