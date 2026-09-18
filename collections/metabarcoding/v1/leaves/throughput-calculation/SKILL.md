---
name: throughput-calculation
description: Use when comparing the processing speed of FASTQ preprocessors (e.g.
  fastp, FASTQC, Cutadapt, SOAPnuke, AfterQC, Trimmomatic, umis, UMI-tools) on the
  same dataset and you need a defensible throughput table.
when_to_use_negative:
- Input is not FASTQ preprocessing (e.g. alignment or variant calling throughput)
  — the published ratios apply to FASTQ preprocessing tools only
- The goal is to compare output quality/filtering behavior rather than speed; time
  measurements alone say nothing about result equivalence
- Datasets or threading configurations differ between tools, making wall-clock times
  incomparable
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Benchmarked FASTQ preprocessor; fastest tool, run in SE/PE single-thread mode
    and with UMI preprocessing (-U --umi_loc=read1)
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Second-fastest comparison tool, run on the same B17NCB1 dataset for timing
- name: Cutadapt
  role: Benchmarked comparison tool, 3-5x slower than fastp
- name: SOAPnuke
  role: Benchmarked comparison tool in SE and PE modes
- name: AfterQC
  role: Benchmarked comparison tool (PE 57.2 min on B17NCB1)
- name: Trimmomatic
  role: Java single-thread benchmarked comparison tool (PE 60.9 min)
- name: umis
  role: UMI preprocessing speed competitor; shifts UMI to read name (12.4 min on 4
    Gb PE150)
- name: UMI-tools
  role: UMI preprocessing speed competitor; slowest (28.3 min on 4 Gb PE150)
- name: Python
  role: Used to compute throughputs and assemble benchmark tables (e.g. benchmark_results.csv)
  repo: https://github.com/OpenGene/fastp/blob/master/parallel.py
merged_aliases:
- sequencing-throughput-calculation
merged_alias_records:
- alias: sequencing-throughput-calculation
  slug: sequencing-throughput-calculation
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_005
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:fa739a681b5b1f0582f2c00b7af5b267475d9905d7681a6433372b93ee449854
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 16
---

# throughput-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark FASTQ preprocessing tool throughput by recording wall-clock execution times on a fixed dataset and converting them to reads/second or GB/minute, as done to show fastp is 2-5x faster than Trimmomatic/Cutadapt and ~2.7x faster than umis and ~6.1x faster than UMI-tools for UMI preprocessing.

## When to use

Use when comparing the processing speed of FASTQ preprocessors (e.g. fastp, FASTQC, Cutadapt, SOAPnuke, AfterQC, Trimmomatic, umis, UMI-tools) on the same dataset and you need a defensible throughput table.

## When NOT to use

- Input is not FASTQ preprocessing (e.g. alignment or variant calling throughput) — the published ratios apply to FASTQ preprocessing tools only
- The goal is to compare output quality/filtering behavior rather than speed; time measurements alone say nothing about result equivalence
- Datasets or threading configurations differ between tools, making wall-clock times incomparable

## Inputs

- Paired-end B17NCB1 FASTQ dataset (9316 M bases), R1/R2 gzip FASTQ files
- 4 Gb Illumina PE150 gzip-compaired paired-end FASTQ with UMI at head of read1
- Tool command lines and thread settings (e.g. fastp -w 1, Trimmomatic single-thread Java)
- Total read counts of the input FASTQ

## Outputs

- benchmark_results.csv with per-tool, per-mode execution times and throughput (reads/s or GB/min)
- Table 1-style six-tool SE/PE speed comparison table (12 tool-by-mode rows)
- Table 3-style UMI preprocessing benchmark (fastp 4.6 min, umis 12.4 min, UMI-tools 28.3 min)

## How to apply

Fix a single benchmark dataset and configuration before running anything: e.g. the paired-end B17NCB1 FASTQ set (9316 M bases) for the six-tool comparison, or 4 Gb Illumina PE150 gzip FASTQ (R1/R2) with the UMI at the head of read1 for the UMI preprocessing comparison. Run each tool on identical input in both SE and PE modes where applicable, controlling threading (fastp -w 1 for single-thread comparisons; default worker count only when the benchmark targets fastp's multithreaded UMI mode). Record wall-clock time per run with a timer, and capture tool reports (fastp.html/fastp.) to confirm the run completed. Convert each recorded time into throughput — reads/second (total reads / time, e.g. fastp PE 13.3 min ≈ 116750 reads/s) or GB of input per minute for the UMI benchmark — and assemble all tool-by-mode rows into a CSV benchmark table mirroring the paper's Table 1/Table 3. Judge correctness by checking the ordering and magnitudes reproduce the published ratios: FASTQC about twice as slow as fastp with others 3-5x slower; umis ~12.4 min and UMI-tools ~28.3 min vs fastp ~4.6 min on 4 Gb PE150 UMI data.

## Related tools

- **fastp** (Benchmarked FASTQ preprocessor; fastest tool, run in SE/PE single-thread mode and with UMI preprocessing (-U --umi_loc=read1)) — https://github.com/OpenGene/fastp
- **FASTQC** (Second-fastest comparison tool, run on the same B17NCB1 dataset for timing)
- **Cutadapt** (Benchmarked comparison tool, 3-5x slower than fastp)
- **SOAPnuke** (Benchmarked comparison tool in SE and PE modes)
- **AfterQC** (Benchmarked comparison tool (PE 57.2 min on B17NCB1))
- **Trimmomatic** (Java single-thread benchmarked comparison tool (PE 60.9 min))
- **umis** (UMI preprocessing speed competitor; shifts UMI to read name (12.4 min on 4 Gb PE150))
- **UMI-tools** (UMI preprocessing speed competitor; slowest (28.3 min on 4 Gb PE150))
- **Python** (Used to compute throughputs and assemble benchmark tables (e.g. benchmark_results.csv)) — https://github.com/OpenGene/fastp/blob/master/parallel.py

## Examples

```
/usr/bin/time -v fastp -w 1 -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -U --umi_loc=read1 --umi_len=10
```

## Evaluation signals

- Recorded fastp PE/SE times reproduce the 2-5x speedup over Trimmomatic and Cutadapt reported in the paper
- Six-tool table contains all 12 tool-by-mode (SE/PE) rows and FASTQC is ~2x slower than fastp with others 3-5x slower
- UMI benchmark ordering matches Table 3: fastp fastest (~4.6 min), umis intermediate (~12.4 min), UMI-tools slowest (~28.3 min)
- Computed fastp PE throughput lands near 116750 reads/s for the 13.3 min run on B17NCB1
- Each tool emits a complete UMI-shifted output FASTQ without error, confirming timed runs were valid

## Limitations

- fastp's speed advantage depends on its C++ implementation with multi-threading; single-threaded comparisons (fastp -w 1) are needed for like-for-like timing against Python/Java tools
- Throughput figures are dataset- and hardware-dependent; ratios (2-5x, 2.7x, 6.1x) rather than absolute times are the transferable result
- The UMI speed benchmark applies to UMI located at the head of read1; other UMI configurations were not benchmarked

## Evidence
