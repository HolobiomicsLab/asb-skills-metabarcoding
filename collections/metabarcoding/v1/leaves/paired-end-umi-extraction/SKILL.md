---
name: paired-end-umi-extraction
description: Use when illumina PE150 FASTQ data carries UMI bases at the head of read1
  and they must be extracted to the read name, especially when preprocessing throughput
  matters.
when_to_use_negative:
- Input is single-end or the UMI is not located at the head of read1 (e.g. inline
  in read2 or in a separate index read), since the benchmarked fastp flags and patterns
  assume read1-head UMIs
- Input is long-read data (Nanopore/PacBio), where the fastp repo directs users to
  fastplong instead
- Goal is UMI-based deduplication or consensus rather than simply shifting the UMI
  to the read name — the compared tools here only extract
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: 'Primary tool: enables UMI preprocessing with -U to shift the UMI to the sequence
    name; fastest in the benchmark (4.6 min on 4 Gb PE150)'
  repo: https://github.com/OpenGene/fastp
- name: umis
  role: 'Comparator: `umis extract` shifts the UMI to the read name on the same paired-end
    input; intermediate speed (12.4 min)'
- name: UMI-tools
  role: 'Comparator: `UMI-tools extract` with the read1 UMI pattern; slowest in the
    benchmark (28.3 min)'
- name: Python
  role: Used to compute throughput (GB processed per minute) and assemble the benchmark
    table
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:05ab1d572fd36fbdd3eea54dc536990442cb482300e7ca69a162ce03b17bc5e2
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
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.1
n_claims: 10
---

# paired-end-umi-extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarked workflow for shifting UMIs from read sequences to read names on paired-end FASTQ data, comparing the wall-clock speed of fastp's UMI preprocessing (-U) against umis extract and UMI-tools extract.

## When to use

Use when Illumina PE150 FASTQ data carries UMI bases at the head of read1 and they must be extracted to the read name, especially when preprocessing throughput matters.

## When NOT to use

- Input is single-end or the UMI is not located at the head of read1 (e.g. inline in read2 or in a separate index read), since the benchmarked fastp flags and patterns assume read1-head UMIs
- Input is long-read data (Nanopore/PacBio), where the fastp repo directs users to fastplong instead
- Goal is UMI-based deduplication or consensus rather than simply shifting the UMI to the read name — the compared tools here only extract

## Inputs

- gzip-compressed Illumina paired-end FASTQ files (R1/R2), ~4 Gb PE150
- UMI located at the head of read1
- UMI length specification (for fastp --umi_len)

## Outputs

- UMI-shifted paired FASTQ (UMI moved to the read name) per tool
- benchmark_results.csv with wall-clock execution times and GB-per-minute throughput per tool

## How to apply

Load the gzip-compressed paired-end FASTQ (R1/R2) and confirm the UMI sits at the head of read1. Run fastp with UMI preprocessing enabled (-U with --umi_loc=read1 and the correct --umi_len) reading the gzipped FASTQ and writing uncompressed output, recording wall-clock time; fastp's UMI feature shifts the UMI into the sequence name in a single scan and is multi-threaded in C++. Run `umis extract` on the same input to shift the UMI to the read name, and `UMI-tools extract` with the read1 UMI pattern, recording execution times for both. Compute GB-per-minute throughput and assemble benchmark_results.csv; on 4 Gb PE150 data expect fastp ~4.6 min, umis ~12.4 min (2.7x slower), UMI-tools ~28.3 min (6.1x slower). Validation: the recorded ordering and magnitudes must reproduce Table 3 and each tool must emit a UMI-shifted FASTQ without error.

## Related tools

- **fastp** (Primary tool: enables UMI preprocessing with -U to shift the UMI to the sequence name; fastest in the benchmark (4.6 min on 4 Gb PE150)) — https://github.com/OpenGene/fastp
- **umis** (Comparator: `umis extract` shifts the UMI to the read name on the same paired-end input; intermediate speed (12.4 min))
- **UMI-tools** (Comparator: `UMI-tools extract` with the read1 UMI pattern; slowest in the benchmark (28.3 min))
- **Python** (Used to compute throughput (GB processed per minute) and assemble the benchmark table)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq -O out.R2.fq -U --umi_loc=read1 --umi_len=10
```

## Evaluation signals

- Recorded execution times reproduce Table 3's ordering: fastp fastest (~4.6 min), umis intermediate (~12.4 min), UMI-tools slowest (~28.3 min)
- Ratios match reported magnitudes: fastp ~2.7x faster than umis and ~6.1x faster than UMI-tools
- Each tool emits a UMI-shifted FASTQ without error (UMI present in the sequence name, absent from the read bases)
- Throughput values (GB input per minute) are consistent with the recorded wall-clock times and 4 Gb input size
- Output FASTQ remains paired (R1/R2 read counts and names still correspond after processing)

## Limitations

- Timings depend on hardware and thread count; the fastp run recorded here used the default worker thread count, so absolute times may shift on other systems even if the ordering holds
- The comparison covers only UMI extraction speed; it does not benchmark the quality/adapter filtering features of the tools
- fastp is designed for short reads (Illumina/MGI); long-read UMI processing falls outside this workflow

## Evidence
