---
name: jaccard-overlap-of-shared-otu-identifications
description: Use when the same amplicon samples have been sequenced or processed in
  two independent runs and you need to quantify how consistently a tool (MAPseq, VSEARCH,
  NINJA-OPS, USEARCH) assigns the same OTUs per sample.
when_to_use_negative:
- Input is already a single-run dataset with no replicate sequencing run — the two-run
  Jaccard overlap is undefined without a paired second run
- The goal is to compare abundance profiles rather than OTU presence/absence sets
  — use the Spearman correlation of OTU abundances across samples instead
- Reads are shotgun metagenomic and OTU clustering is inappropriate — MAPseq supports
  shotgun, but OTU-set Jaccard reproducibility is designed for rRNA amplicon data
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MAPseq 1.2.6
  role: Produces the hierarchical OTU mappings and .mseq classifications from each
    run's rRNA reads; the -otutable option generates the multi-sample OTU count table
    from which shared-OTU sets are extracted for Jaccard computation
  repo: https://github.com/jfmrod/mapseq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:f28ed3e95e7d3768d1d5988aff56c4f675508e40d7b34317117bbfd6e6d5420e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - MAPseq 1.2.6
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Jaccard overlap of shared OTU identifications

## Summary

A cross-run reproducibility metric that computes the per-sample Jaccard index of shared OTU identifications between two independent sequencing runs of the same samples, reporting the median per classification tool (e.g. MAPseq median = 0.52 on the Human Microbiome Project data). It captures which OTUs a tool consistently identifies, independent of abundance.

## When to use

Use when the same amplicon samples have been sequenced or processed in two independent runs and you need to quantify how consistently a tool (MAPseq, VSEARCH, NINJA-OPS, USEARCH) assigns the same OTUs per sample.

## When NOT to use

- Input is already a single-run dataset with no replicate sequencing run — the two-run Jaccard overlap is undefined without a paired second run
- The goal is to compare abundance profiles rather than OTU presence/absence sets — use the Spearman correlation of OTU abundances across samples instead
- Reads are shotgun metagenomic and OTU clustering is inappropriate — MAPseq supports shotgun, but OTU-set Jaccard reproducibility is designed for rRNA amplicon data

## Inputs

- Demultiplexed, quality-filtered rRNA amplicon FASTA/FASTQ files from two independent sequencing runs of the same samples (2194 Human Microbiome Project samples)
- MAPseq .mseq classification outputs for each sample from both runs
- OTU count table(s) generated via 'mapseq -otutable' with OTU labels as rows and samples as columns

## Outputs

- Per-sample Jaccard index of shared OTU identifications between run 1 and run 2
- Median Jaccard value per tool (MAPseq expected median = 0.52)
- Aggregated reproducibility metrics table hmp_reproducibility_metrics.csv ordered by tool

## How to apply

Run MAPseq 1.2.6 (and comparison tools) on every sample from both sequencing runs to obtain .mseq files with hierarchical OTU mappings, then aggregate OTU counts per sample with 'mapseq -otutable'. For each tool, restrict the analysis to OTUs shared between run 1 and run 2 of the same 2194 Human Microbiome Project samples. For each sample compute the Jaccard index between the set of OTU identifications in run 1 and run 2, then record the median Jaccard per tool. Assemble per-tool medians into hmp_reproducibility_metrics.csv ordered by tool. Interpretation: a higher median Jaccard indicates more consistent OTU identification across runs; MAPseq's median of 0.52 (Fig. 1g-h) is the benchmark value to reproduce.

## Related tools

- **MAPseq 1.2.6** (Produces the hierarchical OTU mappings and .mseq classifications from each run's rRNA reads; the -otutable option generates the multi-sample OTU count table from which shared-OTU sets are extracted for Jaccard computation) — https://github.com/jfmrod/mapseq

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq; mapseq -otutable run1_sample.mseq run2_sample.mseq > map.otutable
```

## Evaluation signals

- All computed Jaccard values fall within [0, 1]
- Reproduced table hmp_reproducibility_metrics.csv matches the reported result: MAPseq has the highest median Jaccard of 0.52 (Fig. 1g-h), outperforming VSEARCH, NINJA-OPS, and USEARCH
- Per-sample Jaccard is computed only over OTUs shared between the two runs, as specified in the workflow
- The output table contains one median Jaccard row per tool and is ordered by tool
- MAPseq version is pinned to 1.2.6 (24 Mar 2020) so the '-otutable' fix (v1.2.6: fixed option using only counts from first sample) is active

## Limitations

- Earlier MAPseq releases have a known bug: the '-otutable' option used only counts from the first sample (fixed in v1.2.6), which would invalidate multi-sample OTU tables
- The metric evaluates OTU identification consistency (presence/absence) only; abundance consistency is assessed separately via Spearman correlation
- Only OTUs shared between the two runs are considered, so OTUs detected in a single run are excluded from the comparison
- MAPseq classification confidences depend on the reference dataset and the recommended combined confidence cutoff of 0.4–0.5, which affects which OTUs are counted as identified

## Evidence

- [results] MAPseq achieved the highest cross-run Jaccard overlap of shared OTU identifications (median = 0.52), outperforming VSEARCH, NINJA-OPS, and USEARCH: [evidence span withheld — anchor status: not_found]
- [readme] OTU count table for multiple samples: [evidence span withheld — anchor status: external_doc]
- [readme] v1.2.6 otutable fix: [evidence span withheld — anchor status: external_doc]
- [readme] recommended confidence cutoff: [evidence span withheld — anchor status: external_doc]
