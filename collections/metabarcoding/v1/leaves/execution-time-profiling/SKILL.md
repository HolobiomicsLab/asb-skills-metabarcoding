---
name: execution-time-profiling
description: Use when comparing the runtime performance of competing FASTQ preprocessing
  tools (e.g. UMI extraction by fastp, umis, UMI-tools) on an identical input file,
  and you need reproducible wall-clock timings and throughput numbers.
when_to_use_negative:
- The input is long-read data (Nanopore, PacBio, Cyclone) — fastp targets short reads
  and fastplong should be considered instead.
- The goal is correctness/functional comparison of preprocessing results rather than
  speed benchmarking.
- The FASTQ has no UMI or the UMI is not at a fixed position in read1, so the tools
  cannot be configured for an equivalent task.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Fastest benchmarked tool; performs UMI preprocessing (-U --umi_loc=read1 --umi_len)
    shifting the UMI to the sequence name in a single scan, timed at 4.6 min on 4
    Gb PE150 data
  repo: https://github.com/OpenGene/fastp
- name: umis
  role: Python UMI extraction benchmark; umis extract shifts the UMI to the read name,
    timed at 12.4 min
- name: UMI-tools
  role: UMI extraction benchmark run with the read1 UMI pattern, slowest at 28.3 min
- name: Python
  role: High-level language cited as the reason most other FASTQ preprocessors are
    slower; used for throughput computation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:6586e5eb395035be6c7a193d10f8fd28100104354e2c9300d4ae47a91573daa0
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
  - Python
  - UMI-tools
  - fastp
  - umis
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# execution-time-profiling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark wall-clock execution time of FASTQ preprocessing tools on the same input to quantify speed differences — here, fastp vs umis vs UMI-tools for UMI extraction on 4 Gb Illumina PE150 data, where fastp is ~2.7x faster than umis and ~6.1x faster than UMI-tools.

## When to use

Use when comparing the runtime performance of competing FASTQ preprocessing tools (e.g. UMI extraction by fastp, umis, UMI-tools) on an identical input file, and you need reproducible wall-clock timings and throughput numbers.

## When NOT to use

- The input is long-read data (Nanopore, PacBio, Cyclone) — fastp targets short reads and fastplong should be considered instead.
- The goal is correctness/functional comparison of preprocessing results rather than speed benchmarking.
- The FASTQ has no UMI or the UMI is not at a fixed position in read1, so the tools cannot be configured for an equivalent task.

## Inputs

- gzip-compressed Illumina PE150 FASTQ files (R1/R2), ~4 Gb, with UMI at the head of read1
- UMI length (to configure --umi_len and the UMI-tools read1 pattern)

## Outputs

- UMI-shifted FASTQ from each tool (UMI moved to the sequence name)
- benchmark_results.csv with tool name, wall-clock time (fastp 4.6 min, umis 12.4 min, UMI-tools 28.3 min), and GB/min throughput

## How to apply

Hold the input fixed across all tools: a 4 Gb gzip-compressed Illumina PE150 FASTQ (R1/R2) with the UMI located at the head of read1. Run each tool with the minimum configuration needed for the same task — fastp with UMI preprocessing enabled (-U --umi_loc=read1 --umi_len=<UMI length>), umis extract to shift the UMI to the read name, and UMI-tools extract with the read1 UMI pattern — recording wall-clock execution time for each (fastp with default worker threads). Compute throughput as GB of input processed per minute and assemble a benchmark table (benchmark_results.csv) mirroring Table 3: fastp 4.6 min, umis 12.4 min, UMI-tools 28.3 min. Rationale: fastp is a C++ multi-threaded tool doing a single scan of the FASTQ, whereas competing preprocessors are written in high-level languages (e.g. Python and Java), which explains the measured 2.7–6.1x speed gap.

## Related tools

- **fastp** (Fastest benchmarked tool; performs UMI preprocessing (-U --umi_loc=read1 --umi_len) shifting the UMI to the sequence name in a single scan, timed at 4.6 min on 4 Gb PE150 data) — https://github.com/OpenGene/fastp
- **umis** (Python UMI extraction benchmark; umis extract shifts the UMI to the read name, timed at 12.4 min)
- **UMI-tools** (UMI extraction benchmark run with the read1 UMI pattern, slowest at 28.3 min)
- **Python** (High-level language cited as the reason most other FASTQ preprocessors are slower; used for throughput computation)

## Examples

```
time fastp -i R1.fq.gz -I R2.fq.gz -o out.R1.fq -O out.R2.fq -U --umi_loc=read1 --umi_len=10
```

## Evaluation signals

- Recorded times reproduce Table 3's ordering: fastp fastest (~4.6 min), umis intermediate (~12.4 min), UMI-tools slowest (~28.3 min).
- Each tool completes without error and emits a UMI-shifted FASTQ (UMI relocated from read1 sequence to the read name).
- All tools process the identical 4 Gb gzip PE150 input, so timings are directly comparable.
- Computed throughput (GB/min) is consistent with the recorded wall-clock times and the ~2.7x / ~6.1x speed ratios.

## Limitations

- Timings depend on thread count and hardware; the fastp run used default worker threads, so absolute times may shift on other machines (relative ordering is the validation target).
- fastp's speed advantage partly reflects its C++ implementation with multi-threading and SIMD-accelerated compression libraries (isa-l, libdeflate, highway), which other Python/Java tools do not share.
- The benchmark measures UMI preprocessing only; fastp is reported as 2-5x faster than Trimmomatic or Cutadapt generally, but other tools/tasks were not timed in this workflow.

## Evidence

- [results] fastp UMI preprocessing is approximately 2.7 times faster than umis and about 6.1 times faster than UMI-tools (4.6 min vs 12.4 min and 28.3 min on 4 Gb PE150 data).: [evidence span withheld — anchor status: not_found]
