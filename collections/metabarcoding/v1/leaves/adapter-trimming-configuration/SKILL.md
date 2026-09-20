---
name: adapter-trimming-configuration
description: Use when raw paired-end FASTQ reads (e.g. Illumina NA12878 SRR952827)
  must be cleaned of adapters and low-quality bases before alignment and variant calling,
  and a fast all-in-one preprocessor is preferred.
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead, as
  fastp is designed for short reads
- Adapter sequences cannot be auto-detected or non-standard UMI/low-complexity handling
  is required without reviewing fastp's optional flags
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: 'Primary preprocessor: auto-detects adapters, applies default quality/length
    filtering, and per-read sliding-window cutting in a single FASTQ scan'
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Comparator preprocessor configured with comparable LEADING/TRAILING/SLIDINGWINDOW
    quality-pruning settings for benchmarking
- name: Cutadapt
  role: Comparator preprocessor for adapter trimming in the false-positive variant
    benchmark
- name: SOAPnuke
  role: Comparator preprocessor for adapter trimming and filtering in the benchmark
- name: SpeedSeq
  role: Downstream aligner/variant caller used to assess each preprocessor's effect
    on false-positive variant counts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:5238210f77c3373b12fb10d03e79ba28930cb8c906951af67989b9e6e2e2ed5e
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
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# adapter-trimming-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure adapter trimming and quality filtering on paired-end Illumina FASTQ data using fastp, whose adapter sequences are auto-detected and whose default quality filtering (qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15) runs in a single scan and is 2-5x faster than Trimmomatic or Cutadapt.

## When to use

Use when raw paired-end FASTQ reads (e.g. Illumina NA12878 SRR952827) must be cleaned of adapters and low-quality bases before alignment and variant calling, and a fast all-in-one preprocessor is preferred.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — use fastplong instead, as fastp is designed for short reads
- Adapter sequences cannot be auto-detected or non-standard UMI/low-complexity handling is required without reviewing fastp's optional flags

## Inputs

- Raw paired-end FASTQ files (R1/R2, optionally gzip-compressed) from short-read sequencing (e.g. Illumina NovaSeq, MGI)
- Reference genome and curated variant truth set if benchmarking downstream variant calling

## Outputs

- Filtered/trimmed paired-end FASTQ files (out.R1.fq.gz, out.R2.fq.gz)
- fastp JSON and single-page HTML quality-control report profiling before/after filtering (Q20/Q30, adapter contents, duplication)

## How to apply

Run fastp in paired-end mode ('fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz'), letting its automatic adapter detection trim adapters without supplying sequences manually. Keep the default quality filtering (qualified quality Phred 15, unqualified percent limit 40, length_required 15), which also performs per-read sliding-window cutting of low-quality 5'/3' bases similar to Trimmomatic but faster. For comparison benchmarks, preprocess the identical raw pairs with Cutadapt, SOAPnuke, and Trimmomatic using comparable adapter-trimming and sliding-window quality-pruning settings. Feed each preprocessed pair into the same downstream pipeline (e.g. SpeedSeq alignment and variant calling) and intersect callsets with a curated truth set to count false positives; fastp previously yielded the fewest (6708 vs 6942 Cutadapt, 7040 SOAPnuke, 7174 Trimmomatic).

## Related tools

- **fastp** (Primary preprocessor: auto-detects adapters, applies default quality/length filtering, and per-read sliding-window cutting in a single FASTQ scan) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Comparator preprocessor configured with comparable LEADING/TRAILING/SLIDINGWINDOW quality-pruning settings for benchmarking)
- **Cutadapt** (Comparator preprocessor for adapter trimming in the false-positive variant benchmark)
- **SOAPnuke** (Comparator preprocessor for adapter trimming and filtering in the benchmark)
- **SpeedSeq** (Downstream aligner/variant caller used to assess each preprocessor's effect on false-positive variant counts)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- fastp report JSON/HTML shows adapters were detected and trimmed without manual adapter sequence input
- Filtered output passes default quality filtering: qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15
- Reproduced false-positive variant counts match reported values: fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174 (fastp lowest)
- Q20/Q30 and per-base quality curves in the after-filtering report improve relative to the before-filtering profile

## Limitations

- fastp is designed for short reads; for long reads (Nanopore, PacBio) the fastplong tool should be used instead
- polyX tail trimming and low-complexity filtering are disabled by default and must be explicitly enabled (-x / -y) if desired
- Reported false-positive counts are dataset-specific (NA12878 SRR952827 with SpeedSeq) and may vary with other read sets or callers

## Evidence

- [abstract] fastp is 2-5 times faster than other FASTQ preprocessing tools such as Trimmomatic or Cutadapt despite performing far more operations than similar tools.: "fastp is 2-5 times faster than other FASTQ preprocessing tools such as Trimmomatic or Cutadapt despite performing far more operations than similar tools."
- [results] Filtering NA12878 with each tool and calling variants with SpeedSeq showed fastp yielded the fewest false positive variants.: [evidence span withheld — anchor status: not_found]
- [readme] Adapter sequences can be automatically detected, which means you don't have to input the adapter sequences to trim them.: "Adapter sequences can be automatically detected, which means you don't have to input the adapter sequences to trim them."
- [readme] This tool is designed for processing short reads (i.e. Illumina NovaSeq, MGI), if you are looking for tools to process long reads: "This tool is designed for processing short reads (i.e. Illumina NovaSeq, MGI), if you are looking for tools to process long reads"
- [readme] cut low quality bases for per read in its 5' and 3' by evaluating the mean quality from a sliding window (like Trimmomatic but faster): "cut low quality bases for per read in its 5' and 3' by evaluating the mean quality from a sliding window (like Trimmomatic but faster)"
