---
name: preprocessing-tool-result-aggregation
description: Use when several FASTQ filtering/adapter-trimming tools have been run
  on the same input reads and their outputs must be tabulated and compared on a common
  metric, e.g. suspected-adapter residue rate vs allowed mismatches.
when_to_use_negative:
- Input is raw unfiltered FASTQ with no per-tool outputs to aggregate — run the filterers
  first
- Reads are long-read (Nanopore, PacBio) data — the paper directs such data to fastplong,
  not the short-read tool comparison
- Only a single preprocessing tool was used and no cross-tool comparison is needed
edam_operation: http://edamontology.org/operation_3219
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0622
tools:
- name: fastp
  role: Preprocessing tool whose filtered output is scanned for residual 33 bp adapter
    sequences; also the article's subject tool (C++, multi-threading)
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Java-based comparison tool run with ILLUMINACLIP and SLIDINGWINDOW quality
    pruning; its output is aggregated and scored
- name: Cutadapt
  role: Comparison tool run with the 33 bp adapter sequence supplied; its output is
    aggregated and scored
- name: Python
  role: Scripting language used to scan each tool's filtered reads for the adapter
    sequence under a mismatch sweep and tabulate residue rates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:292a889df2bb11d3f47943186d04340a464b45f47bc5a3ba95a0406cfaa46895
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
  - Python
  - Trimmomatic
  - fastp
  edam_operation: http://edamontology.org/operation_3219
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# preprocessing-tool-result-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aggregates the filtered FASTQ outputs of multiple FASTQ preprocessing tools (fastp, Trimmomatic, Cutadapt, AfterQC, SOAPnuke) into a single benchmark table and comparison figure, scoring each tool by residual adapter content across a mismatch-tolerance sweep.

## When to use

Use when several FASTQ filtering/adapter-trimming tools have been run on the same input reads and their outputs must be tabulated and compared on a common metric, e.g. suspected-adapter residue rate vs allowed mismatches.

## When NOT to use

- Input is raw unfiltered FASTQ with no per-tool outputs to aggregate — run the filterers first
- Reads are long-read (Nanopore, PacBio) data — the paper directs such data to fastplong, not the short-read tool comparison
- Only a single preprocessing tool was used and no cross-tool comparison is needed

## Inputs

- Filtered PE FASTQ output files from each preprocessing tool (fastp, Trimmomatic, Cutadapt, AfterQC, SOAPnuke) run on the same NS_PE150 NextSeq input
- The 33 bp adapter reference sequence to search for
- Mismatch tolerance range (e.g. 0-5 mismatches)

## Outputs

- Summary table of adapter residue rate per tool per mismatch tolerance
- Comparison figure (Figure 6-style layout) ranking the five tools across tolerance levels

## How to apply

Run each filterer on the same PE150 NextSeq input: fastp with default quality filtering (qualified_quality_phred 15, unqualified_percent_limit 40%, length_required 15, n_base_limit 5) plus --detect_adapter_for_pe; Trimmomatic with ILLUMINACLIP and SLIDINGWINDOW:4:20 pruning; Cutadapt with the 33 bp adapter sequence supplied; AfterQC and SOAPnuke in default adapter-trimming modes. Then scan every read of each tool's filtered output with a Python script for the 33 bp adapter sequence while sweeping the allowed mismatch count (e.g. 0-5), recording the fraction of reads still containing adapter. Tabulate the residue rates per tool per tolerance into a summary table and plot the cross-tool comparison in the Figure 6 layout. The rationale is that the paper's finding — fastp and Trimmomatic give the best adapter removal, with fastp-filtered data containing no suspected adapters at ≤4 mismatches — only emerges from aggregating per-tool results under one identical scoring procedure.

## Related tools

- **fastp** (Preprocessing tool whose filtered output is scanned for residual 33 bp adapter sequences; also the article's subject tool (C++, multi-threading)) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Java-based comparison tool run with ILLUMINACLIP and SLIDINGWINDOW quality pruning; its output is aggregated and scored)
- **Cutadapt** (Comparison tool run with the 33 bp adapter sequence supplied; its output is aggregated and scored)
- **Python** (Scripting language used to scan each tool's filtered reads for the adapter sequence under a mismatch sweep and tabulate residue rates)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz --detect_adapter_for_pe
```

## Evaluation signals

- The output figure exists and covers all five tools plus the full mismatch-tolerance sweep
- fastp and Trimmomatic rank among the tools with the lowest residual adapter content, consistent with the reported Figure 6 comparison
- fastp-filtered data contains no suspected adapters when four or fewer mismatches are allowed
- Residue rates are monotonically non-decreasing as allowed mismatches increase within each tool
- All five tools were scored by the same Python scanning script against the same 33 bp adapter sequence, so rates are directly comparable

## Limitations

- The comparison metric counts residual adapter presence by sequence search, so it reflects adapter-trimming performance only, not overall read-quality or filtering differences between tools
- fastp's adapter trimming relies on automatic adapter detection for PE data (--detect_adapter_for_pe); performance may differ if adapters are not detectable
- fastp is designed for short reads (Illumina NovaSeq, MGI); long reads require fastplong

## Evidence
