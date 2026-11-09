---
name: samtools-alignment-qc-evaluation
description: Use when comparing alignment outcomes of NGS read sets preprocessed by
  different tools (fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt, raw) to decide
  which preprocessing yields the cleanest BWA-MEM alignments.
when_to_use_negative:
- Input reads are long reads (Nanopore, PacBio, Cyclone) — the README directs long-read
  data to fastplong, and this short-read hg19/BWA-MEM evaluation does not apply
- Data are unpaired single-end FASTQ — the single-read-map metric (reads whose mate
  failed to map) is undefined without paired mates
- The goal is preprocessing itself rather than alignment-QC benchmarking — use fastp's
  filtering steps (quality, length, low-complexity) directly instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: fastp
  role: Produces the filtered paired-end FASTQ inputs; its filtered data must reproduce
    the best alignment metrics (10.6 M mismatches, 71 single-read maps)
  repo: https://github.com/OpenGene/fastp
- name: BWA
  role: Aligns each tool-filtered read pair set to the hg19 reference genome using
    BWA-MEM
- name: Samtools
  role: Sorts, indexes, and evaluates alignments (flagstat/stats) to extract mapped
    bases, mismatched bases, clips, and single-read maps
- name: Trimmomatic
  role: Alternative FASTQ preprocessor whose filtered output is benchmarked against
    fastp in the comparison
- name: Cutadapt
  role: Alternative FASTQ preprocessor whose filtered output is benchmarked against
    fastp in the comparison
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:d6a92a68e75f69b3126fa7728f57de4e8fd177614cd149473e5088b9a80a5e16
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# samtools-alignment-qc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that aligns tool-filtered paired-end FASTQ data to a reference genome (hg19) with BWA-MEM and evaluates each alignment with Samtools flagstat/stats to compare mismatches, clipped reads, and single-read maps across preprocessing tools.

## When to use

Use when comparing alignment outcomes of NGS read sets preprocessed by different tools (fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt, raw) to decide which preprocessing yields the cleanest BWA-MEM alignments.

## When NOT to use

- Input reads are long reads (Nanopore, PacBio, Cyclone) — the README directs long-read data to fastplong, and this short-read hg19/BWA-MEM evaluation does not apply
- Data are unpaired single-end FASTQ — the single-read-map metric (reads whose mate failed to map) is undefined without paired mates
- The goal is preprocessing itself rather than alignment-QC benchmarking — use fastp's filtering steps (quality, length, low-complexity) directly instead

## Inputs

- gzip-compressed paired-end filtered FASTQ files per preprocessing tool (in.R1.fq.gz / in.R2.fq.gz)
- unfiltered raw NS_PE150 paired-end FASTQ reads (baseline)
- hg19 reference genome FASTQ/FASTA index for BWA-MEM

## Outputs

- per-tool BAM alignments sorted and indexed from BWA-MEM output
- Samtools flagstat/stats metric extracts (mapped bases, mismatched bases, clipped reads, single-read maps)
- table2_mapping_stats.tsv — per-tool mapping-quality comparison table

## How to apply

Collect the gzip-compressed paired-end filtered FASTQ outputs (in.R1.fq.gz / in.R2.fq.gz style) from each preprocessing tool plus the unfiltered NS_PE150 raw reads. Align each read pair set to hg19 with BWA-MEM, then sort and index the resulting BAM files. Run Samtools flagstat/stats to extract four metrics: mapped bases, mismatched bases, soft/hard-clipped reads, and reads whose mate failed to map (single-read maps). Tabulate metrics per tool, anchoring the comparison against the raw-data alignment baseline of 19.8 M mismatched bases. The correct end state is a per-tool table (table2_mapping_stats.tsv) in which fastp-filtered data shows the lowest mismatch count (10.6 M), clip reads (0.302 M), and single-read maps (71).

## Related tools

- **fastp** (Produces the filtered paired-end FASTQ inputs; its filtered data must reproduce the best alignment metrics (10.6 M mismatches, 71 single-read maps)) — https://github.com/OpenGene/fastp
- **BWA** (Aligns each tool-filtered read pair set to the hg19 reference genome using BWA-MEM)
- **Samtools** (Sorts, indexes, and evaluates alignments (flagstat/stats) to extract mapped bases, mismatched bases, clips, and single-read maps)
- **Trimmomatic** (Alternative FASTQ preprocessor whose filtered output is benchmarked against fastp in the comparison)
- **Cutadapt** (Alternative FASTQ preprocessor whose filtered output is benchmarked against fastp in the comparison)

## Examples

```
bwa mem hg19.fa in.R1.fq.gz in.R2.fq.gz | samtools sort -o aln.bam && samtools index aln.bam && samtools flagstat aln.bam
```

## Evaluation signals

- fastp-preprocessed alignment reproduces the reported Table 2 values within tolerance: 3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, 71 single-read maps
- fastp's mismatched-base count (10.6 M) is substantially lower than the raw-data baseline (19.8 M mismatched bases)
- fastp-filtered data has the lowest values across all three metrics (mismatches, clips, single-read maps) relative to AfterQC, SOAPnuke, Trimmomatic, Cutadapt, and raw
- table2_mapping_stats.tsv exists and contains one row of metrics per preprocessing tool plus the raw baseline
- every read set was aligned with the same mapper (BWA-MEM) and evaluated with the same Samtools statistics so the comparison is like-for-like

## Limitations

- Metrics are specific to the NS_PE150 dataset and the hg19 reference; absolute values (e.g. 19.8 M baseline mismatches) do not transfer to other datasets or genomes
- Single-read-map counts depend on paired-end structure and filtering-induced orphaning, so they are only meaningful for paired data
- fastp targets short reads (Illumina NovaSeq, MGI); the README notes long-read processing requires fastplong, so conclusions do not extend to long-read data

## Evidence

- [other] Align each read pair set to the hg19 reference genome using BWA-MEM: [evidence span withheld — anchor status: not_found]
- [other] Sort, index, and evaluate the resulting alignments with Samtools (flagstat/stats): [evidence span withheld — anchor status: not_found]
- [other] confirm the fastp-preprocessed alignment reproduces the reported Table 2 values: [evidence span withheld — anchor status: not_found]
