---
name: adapter-sequence-mismatch-tolerant-search
description: Use when you must quantify how much adapter sequence remains in filtered
  FASTQ output from tools like fastp, Trimmomatic, Cutadapt, AfterQC or SOAPnuke,
  or compare adapter-removal quality across tools on PE150 data.
when_to_use_negative:
- Input reads are raw unfiltered FASTQ — the skill measures adapter residue after
  filtering, not adapter content before trimming
- The goal is adapter removal itself rather than benchmarking how well already-produced
  filtered outputs were trimmed
- Data are long reads (Nanopore, PacBio) — use fastplong-style preprocessing instead
  of this short-read PE150 benchmark
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3572
tools:
- name: fastp
  role: Produces one of the benchmarked filtered FASTQ outputs (with --detect_adapter_for_pe
    and default quality filtering) and is the top-performing tool under the residue
    search; also the article's subject tool
  repo: https://github.com/OpenGene/fastp
- name: Trimmomatic
  role: Java-based filterer producing a benchmarked filtered output using ILLUMINACLIP
    plus SLIDINGWINDOW:4:20 quality pruning
- name: Cutadapt
  role: Filterer producing a benchmarked filtered output when supplied the 33 bp adapter
    sequence
- name: Python
  role: Implements the mismatch-tolerant adapter search script that scans every read
    and tabulates residue rates per tolerance
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/bty560
    title: 'fastp: an ultra-fast all-in-one FASTQ preprocessor'
  source_content_hash: sha256:34ecd24607d8b97dd9e4f5c73707ec3e7f0ee4c9bed6f3611c69d0bc7a93fb8e
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
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3572
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# adapter-sequence-mismatch-tolerant-search

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Post-filtering adapter residue assessment: scan FASTQ reads from each preprocessing tool's output for known 33 bp adapter sequences while sweeping an allowed-mismatch tolerance (0 through ~5), quantifying residual adapter contamination to benchmark adapter-removal performance.

## When to use

Use when you must quantify how much adapter sequence remains in filtered FASTQ output from tools like fastp, Trimmomatic, Cutadapt, AfterQC or SOAPnuke, or compare adapter-removal quality across tools on PE150 data.

## When NOT to use

- Input reads are raw unfiltered FASTQ — the skill measures adapter residue after filtering, not adapter content before trimming
- The goal is adapter removal itself rather than benchmarking how well already-produced filtered outputs were trimmed
- Data are long reads (Nanopore, PacBio) — use fastplong-style preprocessing instead of this short-read PE150 benchmark

## Inputs

- Filtered paired-end FASTQ files (gzipped R1/R2) produced by each adapter-trimming tool on the same NextSeq PE150 input
- The known 33 bp adapter sequence used as the search query
- Mismatch tolerance range to sweep (e.g. 0-5 allowed mismatches)

## Outputs

- Adapter residue rate per tool per mismatch tolerance (fraction of reads still containing adapter)
- Summary table of residue rates across the five tools and tolerance levels
- Comparison plot (Figure 6-style) of residue rate vs allowed mismatches for all tools

## How to apply

Run each filterer (e.g. fastp with default quality filtering — qualified_quality_phred Q15, unqualified_percent_limit 40%, length_required 15, n_base_limit 5 — plus --detect_adapter_for_pe for PE data; Trimmomatic with ILLUMINACLIP and SLIDINGWINDOW:4:20; Cutadapt with the 33 bp adapter supplied; AfterQC and SOAPnuke in default modes) on the same NextSeq PE150 input FASTQ files. Then, with a Python script, search every read in each tool's output for the 33 bp adapter sequence, sweeping the allowed number of mismatches from exact match (0) through a tolerance range (e.g. 0-5), and record the fraction of reads still containing adapter (adapter residue rate) per tool per tolerance. Tabulate the residue rates and plot the comparison across tools and tolerance levels, in the layout of the paper's Figure 6. Rationale: a stricter search (low mismatch tolerance) detects only near-perfect adapter remnants, while higher tolerance catches divergent residues, so the sweep reveals whether a tool's superiority holds across stringency levels — in the reported comparison fastp-filtered data contained no suspected adapters at four or fewer allowed mismatches, with fastp and Trimmomatic ranking best overall.

## Related tools

- **fastp** (Produces one of the benchmarked filtered FASTQ outputs (with --detect_adapter_for_pe and default quality filtering) and is the top-performing tool under the residue search; also the article's subject tool) — https://github.com/OpenGene/fastp
- **Trimmomatic** (Java-based filterer producing a benchmarked filtered output using ILLUMINACLIP plus SLIDINGWINDOW:4:20 quality pruning)
- **Cutadapt** (Filterer producing a benchmarked filtered output when supplied the 33 bp adapter sequence)
- **Python** (Implements the mismatch-tolerant adapter search script that scans every read and tabulates residue rates per tolerance)

## Examples

```
fastp -i in.R1.fq.gz -I in.R2.fq.gz -o out.R1.fq.gz -O out.R2.fq.gz --detect_adapter_for_pe
```

## Evaluation signals

- The residue-rate table covers all five tools (fastp, Trimmomatic, Cutadapt, AfterQC, SOAPnuke) and the full mismatch tolerance sweep (e.g. 0-5)
- The output comparison plot exists and reproduces the Figure 6 layout in which fastp and Trimmomatic show the best adapter removal
- fastp-filtered data shows zero suspected adapters when four or fewer mismatches are allowed, consistent with the reported finding
- All tools were run on the same input PE150 FASTQ files, so residue rates are directly comparable
- Per-tool, per-tolerance residue fractions are monotonically non-decreasing as allowed mismatches increase (sanity check on the search implementation)

## Limitations

- The search depends on knowing the exact 33 bp adapter sequence; results are only as good as the adapter query supplied (fastp can auto-detect adapters, but the benchmark scan uses a fixed query)
- Benchmark reflects one dataset (Illumina NextSeq PE150) and one tolerance range; rankings may differ for other library types or insert-size distributions
- High mismatch tolerance will match partially divergent sequences that may not be true adapter remnants, inflating residue rates

## Evidence

- [other] How do fastp, AfterQC, SOAPnuke, Trimmomatic and Cutadapt compare in adapter-removal performance on the Illumina NextSeq PE150 dataset, as measured by searching 33 bp-long adapter sequences in post-filtering data with mismatch tolerance?: [evidence span withheld — anchor status: not_found]
- [results] fastp and Trimmomatic give the best adapter-removal performance; the fastp-filtered data contains no suspected adapters when four or fewer mismatches are allowed.: [evidence span withheld — anchor status: not_found]
- [methods] cut adapters. Adapter sequences can be automatically detected, which means you don't have to input the adapter sequences to trim them.: "cut adapters. Adapter sequences can be automatically detected, which means you don't have to input the adapter sequences to trim them."
- [readme] trim polyG in 3' ends, which is commonly seen in NovaSeq/NextSeq data.: "trim polyG in 3' ends, which is commonly seen in NovaSeq/NextSeq data."
- [abstract] fastp is 2-5 times faster than other FASTQ preprocessing tools such as Trimmomatic or Cutadapt despite performing far more operations than similar tools.: "fastp is 2-5 times faster than other FASTQ preprocessing tools such as Trimmomatic or Cutadapt despite performing far more operations than similar tools."
