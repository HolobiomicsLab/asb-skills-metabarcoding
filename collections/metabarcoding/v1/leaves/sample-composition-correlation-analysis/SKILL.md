---
name: sample-composition-correlation-analysis
description: Use when you need to quantify how faithfully a metagenomic classifier's
  estimated taxon abundances match known sample composition, e.g. comparing default
  vs memory-efficient MetaMaps runs on mock communities.
when_to_use_negative:
- Samples with no known or simulated ground-truth composition exist, since assignment
  accuracy and composition r2 cannot be computed without reference truth
- Input is short-read metagenomic data with a high proportion of short reads, which
  the article states is not well addressed by MetaMaps' minimizer-based mapping
- The goal is strain/contig-level coverage analysis rather than taxon-frequency correlation
  — use classification_results.EM.contigCoverage instead
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Produces the sample-composition estimates (EMFrequency/PotFrequency in .EM.WIMP)
    and per-read taxon assignments (.EM.reads2Taxon) that are correlated against true
    composition; run in default and memory-efficient (--maxmemory 28/16) modes for
    comparison
  repo: github.com/DiltheyLab/MetaMaps
- name: LomanLab/mockcommunity
  role: Source of real Oxford Nanopore Zymo mock community long-read datasets with
    known composition used for composition-correlation evaluation
  repo: LomanLab/mockcommunity
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:e09d56aa44dbd66c394453d19542d9f96f42b021584b97ce8b4a4f6762167ae4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - LomanLab/mockcommunity
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# sample-composition-correlation-analysis

## Summary

Evaluate taxonomic sample-composition estimates produced by MetaMaps by computing the correlation (r2) between estimated taxon frequencies (e.g. EMFrequency/PotFrequency from the .EM.WIMP output) and true or reference composition, alongside species-level read-assignment accuracy. Used to verify that composition estimation remains accurate (>94% accuracy, r2 > 0.97) across run modes and memory settings.

## When to use

Use when you need to quantify how faithfully a metagenomic classifier's estimated taxon abundances match known sample composition, e.g. comparing default vs memory-efficient MetaMaps runs on mock communities.

## When NOT to use

- Samples with no known or simulated ground-truth composition exist, since assignment accuracy and composition r2 cannot be computed without reference truth
- Input is short-read metagenomic data with a high proportion of short reads, which the article states is not well addressed by MetaMaps' minimizer-based mapping
- The goal is strain/contig-level coverage analysis rather than taxon-frequency correlation — use classification_results.EM.contigCoverage instead

## Inputs

- MetaMaps classification output files: classification_results.EM.WIMP (sample composition), classification_results.EM.reads2Taxon (per-read taxon assignments)
- Ground-truth sample composition for mock communities (e.g. Zymo mock community standards) or simulated read truth assignments
- Long-read FASTQ metagenomic datasets (PacBio/ONT), e.g. GridION/PromethION Zymo mock community runs
- MetaMaps classification database (e.g. miniSeq+H, >12,000 genomes)

## Outputs

- Species-level read-assignment accuracy per experiment and run mode
- Sample-composition correlation (r2) between estimated (PotFrequency/EMFrequency) and true taxon abundances
- Benchmark table (e.g. resource_benchmark_table.csv) with accuracy and r2 per experiment and mode, corresponding to Supplementary Table 1 / Supplementary Data 5

## How to apply

Run MetaMaps on each long-read (PacBio/ONT) metagenomic experiment against a comprehensive RefSeq database such as miniSeq+H (>12,000 genomes), producing classification_results.EM.WIMP and classification_results.EM.reads2Taxon outputs. Parse the .EM.WIMP file, which reports Absolute read counts, EMFrequency (pre-unmapped-correction frequency from the EM algorithm), and PotFrequency (final frequency after correcting for unmapped reads) per taxon. Compute species-level read-assignment accuracy by comparing per-read taxon assignments in .EM.reads2Taxon against ground-truth assignments, and compute the coefficient of determination (r2) between estimated and true sample composition for samples with known composition (e.g. Zymo mock community or simulated data). Repeat the correlation computation for each experimental condition (e.g. default mode vs --maxmemory 28 and --maxmemory 16 memory-efficient mode) and verify accuracy is virtually unaffected; expected reference values are >94% species-level accuracy and r2 > 0.97. Assemble per-experiment accuracy and correlation metrics into a benchmark table (e.g. resource_benchmark_table.csv) for reporting.

## Related tools

- **MetaMaps** (Produces the sample-composition estimates (EMFrequency/PotFrequency in .EM.WIMP) and per-read taxon assignments (.EM.reads2Taxon) that are correlated against true composition; run in default and memory-efficient (--maxmemory 28/16) modes for comparison) — github.com/DiltheyLab/MetaMaps
- **LomanLab/mockcommunity** (Source of real Oxford Nanopore Zymo mock community long-read datasets with known composition used for composition-correlation evaluation) — LomanLab/mockcommunity

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 16; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Species-level read-assignment accuracy exceeds 94% when sample genomes or close relatives are present in the database
- Composition correlation r2 > 0.97 between estimated and true sample composition on simulated and real (mock community) data
- Memory-efficient runs (--maxmemory 28 or 16) show runtime factors of only ~1.2-1.3x (28 GB) and ~1.3-1.4x (16 GB) relative to default mode while assignment accuracy and composition r2 are virtually unchanged
- The .EM.WIMP file parses into three per-taxon columns (Absolute, EMFrequency, PotFrequency) and PotFrequency corrects for unmapped reads
- Final benchmark table (resource_benchmark_table.csv) contains peak RAM, CPU hours, runtime factor, accuracy, and r2 for each experiment x mode combination

## Limitations

- Composition accuracy requires sample genomes or close relatives to be present in the classification database; database-sample incongruity (uncharacterized taxa) is an important unresolved concern
- Complex samples comprising many non-characterized entities remain challenging; detection of novelty from long-read data is an open question
- Analysis of samples with a high proportion of short reads is not well addressed; reads below the minimum length remain unassigned
- No support for streaming data, limiting real-time/clinical use

## Evidence

- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [supplementary] Memory-efficient runtime factor: [evidence span withheld — anchor status: not_found]
- [readme] WIMP output columns definition: [evidence span withheld — anchor status: not_found]
- [intro] EM-based composition estimation step: "each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
