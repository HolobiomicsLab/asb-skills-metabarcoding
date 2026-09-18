---
name: cross-run-abundance-correlation-analysis
description: Use when the same set of samples was sequenced or classified in two independent
  runs and you need to measure how reproducible OTU assignments and abundances are
  across runs.
when_to_use_negative:
- Input is already a single-run dataset with no replicate/second-run sequencing —
  there is no cross-run pairing to correlate
- You need taxonomy-level abundance reproducibility rather than OTU-level (hierarchical
  OTU mappings) identification overlap
- Reads are raw and not demultiplexed/quality filtered — MAPseq expects previously
  demultiplexed and quality-filtered FASTA input
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MAPseq
  role: Primary classifier run (v1.2.6) on every sample from both runs to produce
    taxonomy classifications and hierarchical OTU mappings; its .mseq outputs and
    -otutable option generate the count tables used for the cross-run correlation
    and Jaccard metrics
  repo: https://github.com/jfmrod/mapseq
- name: MAPseq 1.2.6
  role: Pinned MAPseq version used for the two-run consistency analysis on the Human
    Microbiome Project dataset
  repo: https://github.com/jfmrod/mapseq
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:6e74210b4f2e9461aef7bf6ce487e3560669d32e59bc50f70ac072f53635d538
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - MAPseq 1.2.6
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# cross-run-abundance-correlation-analysis

## Summary

Quantifies sequencing-run reproducibility of OTU-level abundance profiles by computing Spearman correlations of shared-OTU abundances and per-sample Jaccard indices of shared OTU identifications between two independent runs of the same samples, as done for the 2194-sample Human Microbiome Project dataset benchmarked against MAPseq, VSEARCH, NINJA-OPS, and USEARCH.

## When to use

Use when the same set of samples was sequenced or classified in two independent runs and you need to measure how reproducible OTU assignments and abundances are across runs.

## When NOT to use

- Input is already a single-run dataset with no replicate/second-run sequencing — there is no cross-run pairing to correlate
- You need taxonomy-level abundance reproducibility rather than OTU-level (hierarchical OTU mappings) identification overlap
- Reads are raw and not demultiplexed/quality filtered — MAPseq expects previously demultiplexed and quality-filtered FASTA input

## Inputs

- FASTA/FASTQ rRNA amplicon reads from each sample in two independent sequencing runs (demultiplexed, quality filtered)
- Per-tool OTU assignments for all reads in both runs (e.g. MAPseq .mseq files or otutable matrices)
- Reference rRNA sequence set with taxonomy/OTU cluster definitions shared across tools

## Outputs

- Per-tool Spearman correlation of shared-OTU abundances across samples between run 1 and run 2
- Per-tool median Jaccard index of shared OTU identifications between runs (e.g. MAPseq median = 0.52)
- Consolidated reproducibility metrics table (hmp_reproducibility_metrics.csv) ordered by tool

## How to apply

Load the paired-run dataset (e.g. the 2194 Human Microbiome Project rRNA amplicon samples sequenced in two runs over different rRNA gene regions), then run MAPseq 1.2.6 and each baseline classifier (VSEARCH, NINJA-OPS, USEARCH) with equivalent reference settings on every sample from both runs to obtain hierarchical OTU mappings. Restrict each tool's comparison to OTUs shared between the two runs and compute the Spearman correlation of OTU abundances across the 2194 samples between run 1 and run 2, capturing the tool's speed-versus-accuracy trade-off. Additionally compute the per-sample Jaccard index of shared OTU identifications between the two runs and record the median per tool. Assemble the per-tool correlation values and median Jaccard values into a single table (hmp_reproducibility_metrics.csv) ordered by tool. In MAPseq, .mseq outputs and the -otutable option are used to build the sample-by-OTU count matrices that feed these metrics; classification confidence can be filtered using the recommended combined confidence cutoff of 0.4 or 0.5.

## Related tools

- **MAPseq** (Primary classifier run (v1.2.6) on every sample from both runs to produce taxonomy classifications and hierarchical OTU mappings; its .mseq outputs and -otutable option generate the count tables used for the cross-run correlation and Jaccard metrics) — https://github.com/jfmrod/mapseq
- **MAPseq 1.2.6** (Pinned MAPseq version used for the two-run consistency analysis on the Human Microbiome Project dataset) — https://github.com/jfmrod/mapseq

## Examples

```
mapseq -otutable run1_s1.mseq run2_s1.mseq > map.otutable
```

## Evaluation signals

- MAPseq achieves the highest median cross-run Jaccard of shared OTU identifications (median = 0.52), outperforming VSEARCH, NINJA-OPS, and USEARCH (Fig. 1g-h)
- All computed Jaccard values fall in the valid interval [0, 1]
- Spearman correlations of shared-OTU abundances are computed only over OTUs present in both runs, across the full 2194-sample set
- hmp_reproducibility_metrics.csv contains one row per tool with correlation and median Jaccard values, ordered by tool
- MAPseq classification uses the recommended combined confidence cutoff of 0.4 or 0.5 that yielded the highest F1/2-score in benchmarks

## Limitations

- The two HMP runs used different rRNA gene regions, so cross-run OTU overlap partly reflects region choice and only OTUs shared between runs contribute to the correlation
- MAPseq data files are searched relative to the binary's path, so moving only the binary breaks the installation
- The -ignoreEmptyTax option (added in v1.2.4) is off by default pending thorough benchmarks, and second hits with missing taxonomic labels can decrease confidence of the top-hit assignment
- Fixed only in v1.2.6: the '-otutable' option previously used only counts from the first sample — earlier versions give wrong multi-sample count tables

## Evidence

- [abstract] MAPseq provides in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences: "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences"
- [readme] OTU table generation for multiple samples: [evidence span withheld — anchor status: external_doc]
- [readme] Recommended confidence cutoff: [evidence span withheld — anchor status: external_doc]
- [readme] Changelog note on -otutable fix: [evidence span withheld — anchor status: external_doc]
