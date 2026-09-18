---
name: benchmark-comparison-plotting
description: Use when several FASTQ preprocessing tools must be compared for adapter-removal
  performance on paired-end Illumina data, and a residual-adapter rate table/figure
  is the deliverable.
when_to_use_negative:
- Input reads are long reads (Nanopore/PacBio/Cyclone) — the README directs such data
  to fastplong, not the short-read fastp pipeline used here
- You need a single tool's QC report only; this skill is specifically for multi-tool
  residue-rate comparison, not routine per-run QC
- The adapter sequences are unknown and cannot be searched for explicitly — the comparison
  metric depends on a defined 33 bp adapter query
edam_operation: http://edamontology.org/operation_0368
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: fastp
  role: Primary PE150 preprocessor; adapter trimming via --detect_adapter_for_pe with
    default quality filtering; reported best residual-adapter performance at <=4 mismatches
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Comparator run in Java with ILLUMINACLIP and SLIDINGWINDOW:4:20 quality pruning;
    ties with fastp for best adapter removal in the comparison
- name: Cutadapt
  role: Comparator run with the 33 bp adapter sequence explicitly supplied
- name: Python
  role: Custom script to scan each filtered FASTQ read for the 33 bp adapter under
    a 0-5 mismatch sweep and tabulate residue rates for plotting
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:056a9b0403466393ed9c26b7a4695a9b26ffe99a65504ee0e2fc5c1f5edbcf05
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
  edam_operation: http://edamontology.org/operation_0368
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# benchmark-comparison-plotting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple FASTQ adapter-trimming/filtering tools (fastp, Trimmomatic, Cutadapt, AfterQC, SOAPnuke) on the same input by scanning post-filtering reads for a 33 bp adapter sequence across a mismatch-tolerance sweep, then tabulate and plot residual adapter rates as a Figure-6-style comparison.

## When to use

Use when several FASTQ preprocessing tools must be compared for adapter-removal performance on paired-end Illumina data, and a residual-adapter rate table/figure is the deliverable.

## When NOT to use

- Input reads are long reads (Nanopore/PacBio/Cyclone) — the README directs such data to fastplong, not the short-read fastp pipeline used here
- You need a single tool's QC report only; this skill is specifically for multi-tool residue-rate comparison, not routine per-run QC
- The adapter sequences are unknown and cannot be searched for explicitly — the comparison metric depends on a defined 33 bp adapter query

## Inputs

- Illumina NextSeq PE150 paired-end FASTQ files (in.R1.fq.gz, in.R2.fq.gz)
- Known 33 bp adapter sequences to search for in post-filtering reads

## Outputs

- Per-tool, per-tolerance adapter residue-rate summary table
- Figure 6-style plot of adapter residue rate across five tools and mismatch tolerance levels (0-5 mismatches)
- Filtered FASTQ outputs from each tool (out.R1.fq.gz, out.R2.fq.gz)

## How to apply

Run fastp on the Illumina NextSeq PE150 data (fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz) with default quality filtering (qualified_quality_phred Q15, unqualified_percent_limit 40%, length_required 15, n_base_limit 5) plus --detect_adapter_for_pe for PE adapter trimming. Run Trimmomatic (Java, ILLUMINACLIP + SLIDINGWINDOW:4:20), Cutadapt (with the 33 bp adapter sequence supplied), and AfterQC/SOAPnuke in default adapter-trimming modes on the same input. For each tool's filtered output, scan every read with a Python script for the 33 bp adapter sequence while sweeping the allowed mismatch count (0 through ~5) and record the fraction of reads still containing adapter. Tabulate adapter residue rates per tool per tolerance, then plot the cross-tool comparison in the Figure 6 layout. Judgment: the figure must cover all five tools and the mismatch sweep, and fastp and Trimmomatic should rank among the lowest residual adapter content.

## Related tools

- **fastp** (Primary PE150 preprocessor; adapter trimming via --detect_adapter_for_pe with default quality filtering; reported best residual-adapter performance at <=4 mismatches) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Comparator run in Java with ILLUMINACLIP and SLIDINGWINDOW:4:20 quality pruning; ties with fastp for best adapter removal in the comparison)
- **Cutadapt** (Comparator run with the 33 bp adapter sequence explicitly supplied)
- **Python** (Custom script to scan each filtered FASTQ read for the 33 bp adapter under a 0-5 mismatch sweep and tabulate residue rates for plotting)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz --detect_adapter_for_pe
```

## Evaluation signals

- Output figure exists and covers all five tools (fastp, AfterQC, SOAPnuke, Trimmomatic, Cutadapt) across the mismatch tolerance sweep (0-5)
- fastp-filtered data shows no suspected adapters when four or fewer mismatches are allowed
- fastp and Trimmomatic rank among the tools with lowest residual adapter content, consistent with the reported Figure 6
- Each tool's filtered output was produced from the identical input FASTQ files, so residue-rate differences are attributable to the tools
- The residue-rate table is populated for every tool/tolerance combination with fractions in [0,1]

## Limitations

- fastp's speed advantage (2-5x vs Trimmomatic/Cutadapt) is about throughput; adapter removal quality must still be assessed empirically via the residue-rate scan
- fastp auto-detects adapter sequences, so results depend on detection working on the given dataset; supplying adapters explicitly (as for Cutadapt) is the alternative
- The mismatch sweep cutoff matters: 'no suspected adapters' for fastp holds only up to four allowed mismatches

## Evidence
