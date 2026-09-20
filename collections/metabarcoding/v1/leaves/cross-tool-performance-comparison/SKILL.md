---
name: cross-tool-performance-comparison
description: 'Use when selecting a FASTQ preprocessing or QC tool on speed grounds:
  you must benchmark wall-clock time and reads/s of fastp vs FASTQC, Cutadapt, SOAPnuke,
  AfterQC and Trimmomatic in SE and PE modes on identical input, all single-threaded.'
when_to_use_negative:
- Input is long-read data (Nanopore/PacBio) — fastp targets short reads (Illumina
  NovaSeq, MGI); fastplong is the long-read counterpart, so this comparison does not
  apply
- The goal is to compare trimming output quality or accuracy rather than speed — timing
  alone cannot judge cleaner reads
- Runs would be multi-threaded or use different input files — this breaks the single-threaded,
  identical-input like-for-like design of the benchmark
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0080
tools:
- name: fastp
  role: Benchmarked tool and fastest reference; run single-threaded (-w 1) in SE and
    PE modes on B17NCB1 to anchor the speed table
  repo: https://github.com/OpenGene/fastp
- name: FASTQC
  role: Comparator QC tool; second-fastest in the benchmark at about twice fastp's
    runtime
- name: Cutadapt
  role: Comparator FASTQ preprocessing tool (high-level language) defining the 2-5x
    speedup bound
- name: SOAPnuke
  role: Comparator preprocessing tool timed in SE and PE modes for the benchmark table
- name: AfterQC
  role: Comparator all-in-one preprocessor; PE run took 57.2 min vs fastp's 13.3 min
- name: Trimmomatic
  role: Comparator tool run single-threaded under Java; slowest PE run (60.9 min),
    bounding the 2-5x speedup claim
- name: Java
  role: Runtime used to execute Trimmomatic single-threaded during the timing runs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:1e50ef92948e87a429367e3d3091065bb19b074fc54b96bdc1c40d576a28e5d4
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
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-tool-performance-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill for FASTQ preprocessing: measure wall-clock processing time and throughput (reads/s) of fastp, FASTQC, Cutadapt, SOAPnuke, AfterQC and Trimmomatic run single-threaded in SE and PE modes on one identical paired-end dataset (B17NCB1, 9316 M bases). It reproduces the paper's Table 1-style comparison showing fastp is 2-5 times faster than tools such as Trimmomatic or Cutadapt.

## When to use

Use when selecting a FASTQ preprocessing or QC tool on speed grounds: you must benchmark wall-clock time and reads/s of fastp vs FASTQC, Cutadapt, SOAPnuke, AfterQC and Trimmomatic in SE and PE modes on identical input, all single-threaded.

## When NOT to use

- Input is long-read data (Nanopore/PacBio) — fastp targets short reads (Illumina NovaSeq, MGI); fastplong is the long-read counterpart, so this comparison does not apply
- The goal is to compare trimming output quality or accuracy rather than speed — timing alone cannot judge cleaner reads
- Runs would be multi-threaded or use different input files — this breaks the single-threaded, identical-input like-for-like design of the benchmark

## Inputs

- Paired-end (and/or single-end) FASTQ files, gzipped or plain (e.g. B17NCB1 R1/R2 FASTQ.gz, 9316 M bases)
- Installed builds of fastp, FASTQC, Cutadapt, SOAPnuke, AfterQC and Trimmomatic (Java runtime), each pinned to single-threaded execution

## Outputs

- CSV benchmark table with 12 tool-by-mode rows (6 tools x SE/PE) holding processing time and throughput (reads/s)
- Per-run wall-clock timing logs plus fastp.html and fastp. reports as provenance artifacts

## How to apply

Obtain one paired-end FASTQ dataset (B17NCB1, 9316 M bases) and run every tool on identical input in both SE mode (fastp -i in.fq -o out.fq) and PE mode (fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz), forcing single-threaded execution (fastp -w 1; Trimmomatic under a single-thread Java run) so the C++ multi-threaded fastp gains no unfair advantage. Record wall-clock processing time for each of the 12 tool-by-mode runs and keep fastp.html/fastp. reports as run artifacts. Compute throughput as total reads processed divided by processing time, using the paper's reference points (fastp PE 13.3 min ≈ 116,750 reads/s; AfterQC PE 57.2 min; Trimmomatic PE 60.9 min). Assemble all SE and PE times and throughputs into a Table 1-style CSV and interpret the ratios against the claim that fastp is 2-5x faster than Trimmomatic/Cutadapt, with FASTQC the second-fastest at roughly twice fastp's runtime.

## Related tools

- **fastp** (Benchmarked tool and fastest reference; run single-threaded (-w 1) in SE and PE modes on B17NCB1 to anchor the speed table) — https://github.com/OpenGene/fastp
- **FASTQC** (Comparator QC tool; second-fastest in the benchmark at about twice fastp's runtime)
- **Cutadapt** (Comparator FASTQ preprocessing tool (high-level language) defining the 2-5x speedup bound)
- **SOAPnuke** (Comparator preprocessing tool timed in SE and PE modes for the benchmark table)
- **AfterQC** (Comparator all-in-one preprocessor; PE run took 57.2 min vs fastp's 13.3 min)
- **Trimmomatic** (Comparator tool run single-threaded under Java; slowest PE run (60.9 min), bounding the 2-5x speedup claim)
- **Java** (Runtime used to execute Trimmomatic single-threaded during the timing runs)

## Examples

```
fastp -w 1 -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- CSV contains all 12 tool-by-mode rows (fastp/FASTQC/Cutadapt/SOAPnuke/AfterQC/Trimmomatic x SE/PE) with both processing-time and reads/s columns populated
- Recomputed throughput = total reads processed / wall-clock time reproduces the reference points: fastp PE ≈ 116,750 reads/s at 13.3 min, AfterQC PE 57.2 min, Trimmomatic PE 60.9 min on B17NCB1
- Ratio check: FASTQC runtime is about 2x fastp and the remaining tools 3-5x fastp, consistent with the paper's 2-5x speedup over Trimmomatic and Cutadapt
- Fairness check: every run used identical input files and single-threaded execution (fastp -w 1, single-thread Java for Trimmomatic); any multi-threaded run invalidates the table
- Per-run fastp artifacts (fastp.html, fastp.) exist, confirming preprocessing completed rather than failed or timed out

## Limitations

- Absolute times are hardware- and dataset-specific (B17NCB1, 9316 M bases); only relative speedups transfer to other setups
- Single-threaded mode understates fastp's real-world speed because it is developed in C++ with multi-threading support
- fastp targets short reads (Illumina NovaSeq, MGI); long reads require fastplong and are out of scope
- Comparators differ in feature scope (e.g. FASTQC is QC-only while fastp performs far more operations in one scan), so equal wall-time would not mean equal work performed
- Trimmomatic and Cutadapt are built with high-level languages (Python, Java), which the paper cites as a source of their slower processing

## Evidence
