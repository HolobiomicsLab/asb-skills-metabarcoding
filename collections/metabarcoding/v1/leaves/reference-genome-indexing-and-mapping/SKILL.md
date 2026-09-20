---
name: reference-genome-indexing-and-mapping
description: 'Decision support for choosing a FASTQ preprocessing tool: you have paired-end
  (PE150) FASTQ files from multiple filter tools plus a raw baseline, and need alignment-based
  metrics (mismatches, clips, single-read maps) against a reference genome to rank
  the filters.'
when_to_use_negative:
- Input is long-read data (Nanopore, PacBio, Cyclone) — the fastp ecosystem explicitly
  redirects such data to fastplong and BWA-MEM is not the appropriate aligner
- Your goal is to assess filtering speed or tool runtime alone; the fastp paper benchmarks
  speed separately (fastp is 2-5x faster than Trimmomatic/Cutadapt) without requiring
  alignment
- You lack a paired reference genome/aligner setup and only need per-file QC; fastp's
  built-in JSON/HTML quality profiling covers before/after filtering without mapping
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BWA
  role: 'Aligner: BWA-MEM maps each tool-filtered read pair set to the hg19 reference
    genome'
- name: Samtools
  role: Sort, index, and evaluate alignments (flagstat/stats) to extract mapped bases,
    mismatched bases, clipped reads, and single-read maps
- name: fastp
  role: Upstream FASTQ preprocessor whose filtered output (adapter trimming, quality
    filtering, per-read sliding-window cutting) is the top-performing input to the
    mapping comparison
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Alternative Java-based FASTQ preprocessing tool whose filtered output is aligned
    as a comparator in the benchmark
- name: Cutadapt
  role: Alternative Python-based adapter-trimming tool whose filtered output is aligned
    as a comparator in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:fc81cb4ddbb93738e393cfe19ff26b306fdcd7544c9509cfd7920adf4766f193
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
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# reference-genome-indexing-and-mapping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Align gzip-compressed paired-end FASTQ reads (e.g. tool-filtered or raw NS_PE150 data) to the hg19 reference genome with BWA-MEM, then sort, index, and evaluate the BAM with Samtools to quantify mapping quality. This skill underpins tool-comparison benchmarks by converting preprocessing quality into measurable alignment statistics.

## When to use

Decision support for choosing a FASTQ preprocessing tool: you have paired-end (PE150) FASTQ files from multiple filter tools plus a raw baseline, and need alignment-based metrics (mismatches, clips, single-read maps) against a reference genome to rank the filters.

## When NOT to use

- Input is long-read data (Nanopore, PacBio, Cyclone) — the fastp ecosystem explicitly redirects such data to fastplong and BWA-MEM is not the appropriate aligner
- Your goal is to assess filtering speed or tool runtime alone; the fastp paper benchmarks speed separately (fastp is 2-5x faster than Trimmomatic/Cutadapt) without requiring alignment
- You lack a paired reference genome/aligner setup and only need per-file QC; fastp's built-in JSON/HTML quality profiling covers before/after filtering without mapping

## Inputs

- gzip-compressed paired-end filtered FASTQ files (in.R1.fq.gz / in.R2.fq.gz) from each preprocessing tool (fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt)
- raw unfiltered NS_PE150 paired-end FASTQ reads (baseline)
- hg19 reference genome FASTA for BWA indexing

## Outputs

- BAM alignment files per tool, sorted and indexed (BAI)
- Samtools flagstat/stats metrics per tool: mapped bases, mismatched bases, soft/hard-clipped reads, single-read maps
- table2_mapping_stats.tsv — per-tool tabulated alignment metrics with raw-data baseline (19.8 M mismatched bases)

## How to apply

Collect the gzip-compressed paired-end filtered FASTQ outputs (in.R1.fq.gz / in.R2.fq.gz style) from each preprocessing tool — fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt — plus the raw unfiltered NS_PE150 reads as baseline. Index the hg19 reference once with BWA, then align each read pair set with BWA-MEM. Sort and index each resulting BAM, and run Samtools flagstat/stats to extract mapped bases, mismatched bases, soft/hard-clipped reads, and reads whose mate failed to map (single-read maps). Tabulate metrics per tool using the raw-data alignment (19.8 M mismatched bases) as the unprocessed baseline; the reference result is fastp's Table 2 profile (3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, 71 single-read maps). The rationale is that a cleaner preprocess (adapter trimming, quality filtering, per-read sliding-window cutting) directly reduces mismatches, clipping, and orphaned mates in the alignment.

## Related tools

- **BWA** (Aligner: BWA-MEM maps each tool-filtered read pair set to the hg19 reference genome)
- **Samtools** (Sort, index, and evaluate alignments (flagstat/stats) to extract mapped bases, mismatched bases, clipped reads, and single-read maps)
- **fastp** (Upstream FASTQ preprocessor whose filtered output (adapter trimming, quality filtering, per-read sliding-window cutting) is the top-performing input to the mapping comparison) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Alternative Java-based FASTQ preprocessing tool whose filtered output is aligned as a comparator in the benchmark)
- **Cutadapt** (Alternative Python-based adapter-trimming tool whose filtered output is aligned as a comparator in the benchmark)

## Examples

```
bwa mem hg19.fa in.R1.fq.gz in.R2.fq.gz | samtools sort -o aln.bam && samtools index aln.bam && samtools flagstat aln.bam > aln.flagstat.txt
```

## Evaluation signals

- fastp-preprocessed alignment reproduces the reported Table 2 values within tolerance: 3102 M mapped bases, 10.6 M mismatched bases, 0.302 M clip reads, 71 single-read maps
- fastp's mismatch count (10.6 M) is substantially lower than the raw-data baseline (19.8 M mismatched bases)
- fastp-filtered data yields the fewest clipped reads (0.302 M) and single-read maps (71) across all compared tools (AfterQC, SOAPnuke, Trimmomatic, Cutadapt, raw)
- Each output BAM is coordinate-sorted and has a valid companion BAI index before Samtools stats is run
- table2_mapping_stats.tsv contains one metric row per preprocessing tool plus the raw baseline

## Limitations

- Alignment-quality ranking is specific to this dataset (NS_PE150, hg19, BWA-MEM); other read types or references may reorder tool rankings
- Samtools stats captures mismatch, clipping, and mate-mapping counts but does not by itself explain why a filter performed better or worse
- fastp's quality/length filtering are enabled by default and low-complexity filtering is disabled by default, so tool-comparison results depend on per-tool default settings unless harmonized

## Evidence

- [other] Align each read pair set to the hg19 reference genome using BWA-MEM.: [evidence span withheld — anchor status: not_found]
- [other] Sort, index, and evaluate the resulting alignments with Samtools (flagstat/stats): [evidence span withheld — anchor status: not_found]
- [results] fastp-filtered data yielded the best alignment quality, with the lowest number of mismatches (10.6 M), clipped reads (0.302 M), and single-read mapped reads (71): [evidence span withheld — anchor status: not_found]
