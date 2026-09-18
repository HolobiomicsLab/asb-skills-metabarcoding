---
name: fastq-umi-shifting-preprocessing
description: Use when illumina paired-end FASTQ data carries a UMI at the head of
  read1 and the UMI must be shifted to the sequence name before downstream deduplication,
  especially when processing time on large (e.g. 4 Gb) files matters.
when_to_use_negative:
- The UMI is not located at the head of read1 (e.g. inline in read2 or in a separate
  index read) — the --umi_loc/umi_len settings here would extract the wrong bases
- Input is long-read data (Nanopore, PacBio) — fastp is designed for short reads and
  the article directs users to fastplong instead
- The data has no UMI and only generic quality/adapter preprocessing is needed — the
  general fastp QC workflow, not this UMI-shifting benchmark skill, applies
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Primary UMI preprocessing tool; enables UMI shifting to the sequence name
    via -U/--umi with --umi_loc and --umi_len, fastest of the compared tools
  repo: https://github.com/OpenGene/fastp
- name: umis
  role: Comparison baseline; 'umis extract' shifts the UMI to the read name from paired-end
    input
- name: UMI-tools
  role: Comparison baseline; 'UMI-tools extract' removes the read1 UMI pattern, slowest
    of the compared tools
- name: Python
  role: Used to assemble the benchmark_results.csv throughput comparison table from
    recorded execution times
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:7377b06c325779502a85dd2954b36df94de7cd3e72ddfaac3f2dbdb83fbd9b54
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# fastq-umi-shifting-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark and execute FASTQ UMI preprocessing (shifting the UMI into the read name) using fastp versus umis and UMI-tools, where fastp is ~2.7x faster than umis and ~6.1x faster than UMI-tools on 4 Gb Illumina PE150 data.

## When to use

Use when Illumina paired-end FASTQ data carries a UMI at the head of read1 and the UMI must be shifted to the sequence name before downstream deduplication, especially when processing time on large (e.g. 4 Gb) files matters.

## When NOT to use

- The UMI is not located at the head of read1 (e.g. inline in read2 or in a separate index read) — the --umi_loc/umi_len settings here would extract the wrong bases
- Input is long-read data (Nanopore, PacBio) — fastp is designed for short reads and the article directs users to fastplong instead
- The data has no UMI and only generic quality/adapter preprocessing is needed — the general fastp QC workflow, not this UMI-shifting benchmark skill, applies

## Inputs

- gzip-compressed Illumina paired-end FASTQ files (R1/R2), e.g. 4 Gb PE150, with UMI located at the head of read1
- UMI length specification (umi_len) matching the library design

## Outputs

- UMI-shifted FASTQ files in which the UMI has been moved to the sequence/read name
- Benchmark results table (benchmark_results.csv) with wall-clock times and throughput (GB/min) per tool, reproducing Table 3 (fastp 4.6 min, umis 12.4 min, UMI-tools 28.3 min)

## How to apply

Load the gzip-compaired paired-end FASTQ (R1/R2) and confirm the UMI location and length at the head of read1. Run fastp with UMI preprocessing enabled (-U --umi_loc=read1 --umi_len=<UMI length>), writing uncompressed output and recording wall-clock time with default worker threads. Optionally run umis extract (shifts UMI to the read name) and UMI-tools extract (with the read1 UMI pattern) on the same input for comparison, recording execution times and computing GB/min throughput for a benchmark table. fastp's speed advantage stems from its C++ implementation with multi-threading, performing UMI shifting in a single scan of the FASTQ data alongside other preprocessing. Validate that each tool emits a UMI-shifted FASTQ without error and that timing orderings are reproduced.

## Related tools

- **fastp** (Primary UMI preprocessing tool; enables UMI shifting to the sequence name via -U/--umi with --umi_loc and --umi_len, fastest of the compared tools) — https://github.com/OpenGene/fastp
- **umis** (Comparison baseline; 'umis extract' shifts the UMI to the read name from paired-end input)
- **UMI-tools** (Comparison baseline; 'UMI-tools extract' removes the read1 UMI pattern, slowest of the compared tools)
- **Python** (Used to assemble the benchmark_results.csv throughput comparison table from recorded execution times)

## Examples

```
fastp -i R1.fq.gz -I R2.fq.gz -o out.R1.fq -O out.R2.fq -U --umi_loc=read1 --umi_len=10
```

## Evaluation signals

- Recorded wall-clock times reproduce Table 3's ordering and approximate magnitudes: fastp ~4.6 min, umis ~12.4 min, UMI-tools ~28.3 min on 4 Gb PE150 data
- Each tool emits a UMI-shifted FASTQ without error, with the UMI sequence present in the read name
- Computed throughput ordering matches: fastp ~2.7x faster than umis and ~6.1x faster than UMI-tools
- Output FASTQ is readable and paired (R1/R2 record counts match) after UMI shifting

## Limitations

- fastp is designed for short reads (Illumina NovaSeq, MGI); long reads require fastplong
- UMI processing must be explicitly enabled with -U/--umi; the UMI location and length must be correctly specified or shifting silently extracts wrong bases
- Benchmark timings are hardware- and thread-count dependent; fastp's multi-threaded C++ implementation contributes to its speed advantage and results vary with default worker thread count

## Evidence
