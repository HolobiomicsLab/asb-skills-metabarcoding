---
name: sliding-window-quality-pruning
description: Use when Illumina FASTQ reads carry low-quality tails or bases that would
  inflate sequencing-error-driven false-positive variant calls, and you need sliding-window
  per-read quality trimming comparable to Trimmomatic but faster.
when_to_use_negative:
- Input reads are long reads (Nanopore, PacBio, Cyclone) — use fastplong instead,
  since fastp is designed for short reads
- Reads are already quality-pruned and adapter-free; re-cutting may over-trim and
  shorten reads unnecessarily
- Low-complexity artifacts rather than quality tails are the main problem — fastp's
  low_complexity_filter is disabled by default and must be explicitly enabled with
  -y
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3673
tools:
- name: fastp
  role: Primary tool performing per-read sliding-window quality cutting (like Trimmomatic
    but faster), plus default quality/length filtering and adapter auto-detection
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Benchmark comparator whose LEADING/TRAILING/SLIDINGWINDOW quality-pruning
    semantics fastp mirrors; yielded 7174 false-positive variants vs fastp's 6708
- name: Cutadapt
  role: Benchmark comparator applied with comparable adapter-trimming settings; 6942
    false-positive variants
- name: SOAPnuke
  role: Benchmark comparator with comparable preprocessing settings; 7040 false-positive
    variants
- name: SpeedSeq
  role: Aligns preprocessed pairs and calls variants used to quantify false positives
    against the NA12878 curated set
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:893c031f8639138f66f50b50e98d441e8856d4d624beb489a60d834897741a86
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
  - http://edamontology.org/topic_3673
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# sliding-window-quality-pruning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-read sliding-window cutting of low-quality 5'/3' bases from FASTQ reads by evaluating mean quality scores in a window, as implemented in fastp. This quality-pruning step is a key reason fastp preprocessing reduced false-positive variant calls (6708 FPs) versus Trimmomatic (7174) in SpeedSeq calling on NA12878.

## When to use

Use when Illumina FASTQ reads carry low-quality tails or leading bases that would inflate sequencing-error-driven false-positive variant calls, and you need sliding-window per-read quality trimming comparable to Trimmomatic but faster.

## When NOT to use

- Input reads are long reads (Nanopore, PacBio, Cyclone) — use fastplong instead, since fastp is designed for short reads
- Reads are already quality-pruned and adapter-free; re-cutting may over-trim and shorten reads unnecessarily
- Low-complexity artifacts rather than quality tails are the main problem — fastp's low_complexity_filter is disabled by default and must be explicitly enabled with -y

## Inputs

- Paired-end Illumina FASTQ files (optionally gzip-compressed), e.g. SRR952827 NA12878 R1/R2 FASTQ
- Reference genome and NA12878 curated variant truth set (for downstream FP evaluation)

## Outputs

- Cleaned paired FASTQ files with sliding-window quality-pruned reads (out.R1.fq.gz, out.R2.fq.gz)
- fastp QC/JSON/HTML report profiling before- and after-filtering quality
- Downstream SpeedSeq variant callsets and false-positive counts (fp_counts.csv)

## How to apply

Run fastp in paired-end mode ('fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz') so per-read cutting evaluates mean quality from a sliding window and trims low-quality bases at both 5' and 3' ends, analogous to Trimmomatic's LEADING/TRAILING/SLIDINGWINDOW. Keep default quality filtering enabled (qualified_quality_phred 15, unqualified_percent_limit 40, length_required 15) and let fastp auto-detect adapter sequences for trimming. For comparability in benchmarking, apply the same adapter-trimming and window-pruning logic with Cutadapt, SOAPnuke, and Trimmomatic on the same raw pairs. Downstream, align each preprocessed pair and call variants with SpeedSeq, then intersect callsets with the NA12878 curated truth set to count false positives and confirm pruning improved specificity.

## Related tools

- **fastp** (Primary tool performing per-read sliding-window quality cutting (like Trimmomatic but faster), plus default quality/length filtering and adapter auto-detection) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Benchmark comparator whose LEADING/TRAILING/SLIDINGWINDOW quality-pruning semantics fastp mirrors; yielded 7174 false-positive variants vs fastp's 6708)
- **Cutadapt** (Benchmark comparator applied with comparable adapter-trimming settings; 6942 false-positive variants)
- **SOAPnuke** (Benchmark comparator with comparable preprocessing settings; 7040 false-positive variants)
- **SpeedSeq** (Aligns preprocessed pairs and calls variants used to quantify false positives against the NA12878 curated set)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz
```

## Evaluation signals

- fastp JSON/HTML report shows per-read quality curves improved after filtering (Q20/Q30 raised, adapter content reduced)
- No output reads fall below the default length_required 15 or qualified_quality_phred 15 / unqualified_percent_limit 40 thresholds
- Reproduced false-positive counts match reported values: fastp 6708, Cutadapt 6942, SOAPnuke 7040, Trimmomatic 7174 (fastp lowest)
- Output FASTQ remains paired: R1/R2 output files have equal read counts (unpaired reads, if kept, go to the unpaired output)

## Limitations

- PolyG tail trimming is NovaSeq/NextSeq-specific and polyX tail trimming is disabled by default (enable with -x), so tailing artifacts may persist unless explicitly enabled
- Low complexity filtering is disabled by default in fastp and must be enabled with -y/--low_complexity_filter
- fastp is designed for short reads (Illumina NovaSeq, MGI); long reads require fastplong

## Evidence
