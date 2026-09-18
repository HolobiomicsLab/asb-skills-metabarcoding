---
name: preprocessing-tool-benchmark-comparison
description: Use when choosing a FASTQ preprocessing/trimming tool for Illumina paired-end
  data and you need an empirical, alignment-based ranking rather than vendor speed
  or feature claims.
when_to_use_negative:
- Input reads are long-read data (Nanopore, PacBio, Cyclone) — fastp targets short
  reads and the authors direct such data to fastplong instead
- Reads are already adapter-trimmed and quality-filtered, so a second preprocessing
  round would only shorten reads further and confound the comparison
- The goal is benchmarking preprocessor runtime/speed only, not downstream alignment
  quality
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: Candidate preprocessing tool whose filtered output produced the best alignment
    metrics (10.6 M mismatches, 0.302 M clip reads, 71 single-read maps)
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Comparator preprocessing tool whose filtered reads are benchmarked against
    fastp-filtered reads
- name: Cutadapt
  role: Comparator preprocessing tool benchmarked in the same alignment-quality evaluation
- name: BWA
  role: BWA-MEM aligns each tool's filtered read pairs to hg19 to expose preprocessing
    quality
- name: Samtools
  role: flagstat/stats on sorted, indexed BWA-MEM BAMs extract mapped bases, mismatches,
    clips, and single-read maps per tool
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:e12f62bc0d650d52a15ebb3ab7c966236611f44c0bc3366c60ac261d8bb1cb05
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
  - BWA
  - Cutadapt
  - Samtools
  - Trimmomatic
  - fastp
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# preprocessing-tool-benchmark-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple FASTQ preprocessing tools (fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt) by aligning each tool's filtered NS_PE150 reads to hg19 with BWA-MEM and scoring alignment cleanliness with Samtools, revealing which tool yields the fewest mismatches, clips, and single-read maps.

## When to use

Use when choosing a FASTQ preprocessing/trimming tool for Illumina paired-end data and you need an empirical, alignment-based ranking rather than vendor speed or feature claims.

## When NOT to use

- Input reads are long-read data (Nanopore, PacBio, Cyclone) — fastp targets short reads and the authors direct such data to fastplong instead
- Reads are already adapter-trimmed and quality-filtered, so a second preprocessing round would only shorten reads further and confound the comparison
- The goal is benchmarking preprocessor runtime/speed only, not downstream alignment quality

## Inputs

- gzip-compressed paired-end filtered FASTQ files from each preprocessing tool (in.R1.fq.gz / in.R2.fq.gz style outputs)
- raw unfiltered NS_PE150 paired-end FASTQ reads (baseline)
- hg19 reference genome FASTA plus BWA index

## Outputs

- Per-tool BWA-MEM alignments (BAM) sorted and indexed
- Samtools flagstat/stats metric tables (mapped bases, mismatched bases, clip reads, single-read maps)
- table2_mapping_stats.tsv with one row per preprocessing tool plus raw baseline

## How to apply

Collect the gzip-compressed paired-end filtered FASTQ outputs (in.R1.fq.gz / in.R2.fq.gz style) from each preprocessing tool plus the raw unfiltered NS_PE150 reads as baseline. Align each read-pair set to hg19 using BWA-MEM, then sort/index the BAMs and run Samtools flagstat/stats to extract four cleanliness metrics: mapped bases, mismatched bases, soft/hard-clipped reads, and reads whose mate failed to map (single-read maps). Tabulate all metrics per tool against the raw-data baseline (19.8 M mismatched bases in this study). Validate by confirming the winning tool's values reproduce the reference table within tolerance — fastp scored 3102 M mapped bases, 10.6 M mismatches, 0.302 M clipped reads, and 71 single-read maps. The rationale is that alignment quality directly exposes whether a preprocessor removed adapters/low-quality bases (quality filtering, per-read sliding-window cutting, adapter auto-detection, PE overlap base correction are all relevant fastp operations) without discarding useful data.

## Related tools

- **fastp** (Candidate preprocessing tool whose filtered output produced the best alignment metrics (10.6 M mismatches, 0.302 M clip reads, 71 single-read maps)) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Comparator preprocessing tool whose filtered reads are benchmarked against fastp-filtered reads)
- **Cutadapt** (Comparator preprocessing tool benchmarked in the same alignment-quality evaluation)
- **BWA** (BWA-MEM aligns each tool's filtered read pairs to hg19 to expose preprocessing quality)
- **Samtools** (flagstat/stats on sorted, indexed BWA-MEM BAMs extract mapped bases, mismatches, clips, and single-read maps per tool)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz -h fastp.html -j fastp.
```

## Evaluation signals

- fastp-filtered alignment reproduces Table 2 values within tolerance: 3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, 71 single-read maps
- fastp mismatch count (10.6 M) is substantially lower than the raw-data baseline (19.8 M mismatched bases)
- Every tool's metric row is present in table2_mapping_stats.tsv, including the raw baseline, so no comparator is silently dropped
- Samtools flagstat/stats were run on sorted and indexed BAMs (sort/index before stats, not on unsorted output)
- Read-pair inputs to the aligner are the gzip-compressed paired files (R1/R2) produced by each tool, keeping the comparison symmetrical across tools

## Limitations

- The evaluation depends on Samtools-derived metrics; preprocessing differences that do not change mismatch/clip/single-read counts will not be distinguished
- fastp's quality and length filtering are enabled by default while low-complexity filtering is off, so default-parameter benchmarking may not reflect all available tool behavior
- fastp is designed for short reads (Illumina NovaSeq, MGI); long-read data would require fastplong and invalidate this benchmark design

## Evidence
