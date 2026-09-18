---
name: paired-end-short-read-alignment
description: Use when you have paired-end FASTQ files (e.g. in.R1.fq.gz/in.R2.fq.gz)
  that must be mapped to a reference genome (hg19) and compared on alignment quality
  metrics across preprocessing variants.
when_to_use_negative:
- Input reads are long reads (Nanopore, PacBio, Cyclone) rather than short Illumina/MGI-style
  paired-end reads — use a long-read preprocessing and alignment workflow instead.
- You need only FASTQ quality control/preprocessing (QC curves, Q20/Q30, adapter content)
  and not genome mapping — a preprocessor like fastp alone suffices.
- Only a single-end dataset with no preprocessing-tool comparison is involved and
  no mismatch/clip/single-read-map metrics are required.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3168
tools:
- name: fastp
  role: Ultra-fast FASTQ preprocessor (quality filtering, adapter trimming, per-read
    sliding-window quality cutting) used upstream to produce the cleanest paired-end
    input for BWA-MEM alignment
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Alternative FASTQ preprocessing tool whose filtered output was aligned and
    compared against fastp's on alignment quality
- name: Cutadapt
  role: Alternative adapter-trimming tool whose filtered output was aligned and compared
    in the tool comparison
- name: BWA
  role: BWA-MEM aligns each filtered (or raw) paired-end read set to the hg19 reference
    genome
- name: Samtools
  role: Sorts, indexes, and evaluates the resulting alignments (flagstat/stats) to
    extract mapped bases, mismatches, clips, and single-read maps
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:970b51fe4368295a086d3a6bcbfffa9011f6f59dc320cd185acd1f5e80195e8b
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# paired-end-short-read-alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align gzip-compressed paired-end FASTQ reads (e.g. NS_PE150 data, preprocessed by tools such as fastp) to a reference genome like hg19 using BWA-MEM, then evaluate alignment quality with Samtools to compare preprocessing tools by mismatched bases, clipped reads, and single-read maps.

## When to use

Use when you have paired-end FASTQ files (e.g. in.R1.fq.gz/in.R2.fq.gz) that must be mapped to a reference genome (hg19) and compared on alignment quality metrics across preprocessing variants.

## When NOT to use

- Input reads are long reads (Nanopore, PacBio, Cyclone) rather than short Illumina/MGI-style paired-end reads — use a long-read preprocessing and alignment workflow instead.
- You need only FASTQ quality control/preprocessing (QC curves, Q20/Q30, adapter content) and not genome mapping — a preprocessor like fastp alone suffices.
- Only a single-end dataset with no preprocessing-tool comparison is involved and no mismatch/clip/single-read-map metrics are required.

## Inputs

- gzip-compressed paired-end FASTQ files (in.R1.fq.gz / in.R2.fq.gz style outputs from preprocessing tools)
- unfiltered raw NS_PE150 paired-end FASTQ reads (baseline)
- hg19 reference genome (indexed for BWA-MEM)

## Outputs

- BAM alignment per read-pair set, sorted and indexed
- Samtools flagstat/stats metrics: mapped bases, mismatched bases, soft/hard-clipped reads, single-read maps
- table2_mapping_stats.tsv with per-tool mapping statistics

## How to apply

Collect the gzip-compressed paired-end filtered FASTQ files produced by each preprocessing tool (e.g. fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt) plus the unfiltered raw NS_PE150 reads as a baseline. Align each read-pair set to the hg19 reference with BWA-MEM, then sort and index the BAM output. Evaluate each alignment with Samtools (flagstat/stats) to extract mapped bases, mismatched bases, soft/hard-clipped reads, and reads whose mate failed to map (single-read maps). Tabulate metrics per tool using the raw-data alignment (19.8 M mismatched bases) as the unprocessed baseline; in the source comparison, fastp-filtered data reproduced Table 2 values of 3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, and 71 single-read maps, showing that clean upstream preprocessing (quality filtering, adapter trimming, per-read sliding-window cutting as done by fastp) yields measurably cleaner BWA-MEM alignments.

## Related tools

- **fastp** (Ultra-fast FASTQ preprocessor (quality filtering, adapter trimming, per-read sliding-window quality cutting) used upstream to produce the cleanest paired-end input for BWA-MEM alignment) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Alternative FASTQ preprocessing tool whose filtered output was aligned and compared against fastp's on alignment quality)
- **Cutadapt** (Alternative adapter-trimming tool whose filtered output was aligned and compared in the tool comparison)
- **BWA** (BWA-MEM aligns each filtered (or raw) paired-end read set to the hg19 reference genome)
- **Samtools** (Sorts, indexes, and evaluates the resulting alignments (flagstat/stats) to extract mapped bases, mismatches, clips, and single-read maps)

## Examples

```
bwa mem hg19.fa in.R1.fq.gz in.R2.fq.gz | samtools sort -o aln.bam; samtools index aln.bam; samtools flagstat aln.bam > flagstat.txt
```

## Evaluation signals

- fastp-preprocessed alignment reproduces the reported Table 2 values (3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, 71 single-read maps) within tolerance
- fastp's mismatch count (10.6 M) is substantially lower than the raw-data baseline (19.8 M mismatched bases)
- fastp-filtered data shows the lowest clip reads (0.302 M) and single-read maps (71) among all tools compared (AfterQC, SOAPnuke, Trimmomatic, Cutadapt)
- Each BAM is successfully sorted and indexed before Samtools evaluation, and table2_mapping_stats.tsv contains one row of metrics per tool plus the raw baseline

## Limitations

- Alignment quality conclusions are tied to the NS_PE150 dataset and hg19 reference; absolute metric values (e.g. 3102 M mapped bases) will not transfer to other datasets.
- fastp is designed for short reads (Illumina NovaSeq, MGI); long-read data require fastplong instead.
- Comparison depends on consistent preprocessing defaults; disabling quality or length filtering (fastp -Q / -L) or enabling others (low-complexity filter -y, polyX trimming -x) changes the filtered output and thus alignment metrics.

## Evidence

- [results] fastp-filtered data yielded the best alignment quality, with the lowest number of mismatches (10.6 M), clipped reads (0.302 M), and single-read mapped reads (71): [evidence span withheld — anchor status: not_found]
- [methods] Align each read pair set to the hg19 reference genome using BWA-MEM.: [evidence span withheld — anchor status: not_found]
