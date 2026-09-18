---
name: multi-level-taxonomic-abundance-aggregation
description: Use when you have classified rRNA amplicon reads (MAPseq .mseq output)
  and need per-sample abundance tables at specific taxonomy levels, or when benchmarking
  how well a classifier recovers known mock community compositions.
when_to_use_negative:
- Input is already a feature/abundance table — this skill starts from raw or MAPseq-classified
  reads, not pre-computed counts.
- Sequences are not rRNA amplicons or lack a suitable full-length rRNA reference with
  known taxonomy — MAPseq is a reference-based rRNA classifier and cannot be applied
  to arbitrary sequence types.
- Mock community has no known expected composition at the target taxonomic levels
  — the accuracy comparison against expected abundances cannot be computed.
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MAPseq
  role: Classifies rRNA reads against reference taxonomy/OTU sets and aggregates per-read
    assignments into multi-level abundance count tables (-otucounts, -otutable).
  repo: https://github.com/jfmrod/mapseq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:ccc5f6c06442417fc85ded0376534720e9b5507ee9591e674e10a06ee02d783c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  - gcc
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# multi-level-taxonomic-abundance-aggregation

## Summary

Aggregates MAPseq rRNA read classifications into abundance profiles at multiple taxonomic ranks (species, genus, family) and compares them against expected compositions, e.g. for artificial 'mock' community benchmarking of classification tools.

## When to use

Use when you have classified rRNA amplicon reads (MAPseq .mseq output) and need per-sample abundance tables at specific taxonomy levels, or when benchmarking how well a classifier recovers known mock community compositions.

## When NOT to use

- Input is already a feature/abundance table — this skill starts from raw or MAPseq-classified reads, not pre-computed counts.
- Sequences are not rRNA amplicons or lack a suitable full-length rRNA reference with known taxonomy — MAPseq is a reference-based rRNA classifier and cannot be applied to arbitrary sequence types.
- Mock community has no known expected composition at the target taxonomic levels — the accuracy comparison against expected abundances cannot be computed.

## Inputs

- FASTA file of demultiplexed, quality-filtered rRNA amplicon reads (e.g. rawseqs.fa)
- MAPseq .mseq classification output files (confidences or simple format)
- Expected taxonomic composition of an artificial mock community (known at species, genus, family level)

## Outputs

- Per-sample count summaries per taxonomy and level (.mseq summary via -otucounts)
- Tab-separated OTU/taxa count table for multiple samples (map.otutable), importable into R
- Tool-by-level comparison metrics between recovered and expected mock community abundances

## How to apply

Classify the demultiplexed, quality-filtered amplicon reads with MAPseq against its reference set, keeping the .mseq 'confidences' output. Aggregate per-read assignments into counts with 'mapseq -otucounts' (single sample) or 'mapseq -otutable' (multi-sample TSV, importable into R), selecting the taxonomy (-ti, e.g. NCBI) and level (-tl, e.g. family/genus/species). Compute agreement metrics between the recovered and expected abundances at each of the species, genus and family levels; the mock community benchmark showed MAPseq recovers expected abundances better than competing tools at all three levels. Use the combined confidence cutoff of 0.4–0.5 when filtering assignments, since this yielded the highest F1/2-score in the authors' benchmarks. Ensure MAPseq v1.2.6 or later so '-otutable' includes counts from all samples, not only the first.

## Related tools

- **MAPseq** (Classifies rRNA reads against reference taxonomy/OTU sets and aggregates per-read assignments into multi-level abundance count tables (-otucounts, -otutable).) — https://github.com/jfmrod/mapseq

## Examples

```
mapseq -otutable sample1.mseq sample2.mseq > map.otutable
```

## Evaluation signals

- The -otucounts output header reports the total read count (e.g. '#sample.mseq 102301'); aggregated label counts should be consistent with the number of classified reads in the .mseq file.
- The -otutable TSV loads cleanly in R via read.table(..., sep='\t', header=TRUE) with samples as columns and taxonomic labels as rows.
- Recovered abundance profiles at species, genus and family levels should agree with the known mock community composition better than competing tools (consistent with MAPseq's reported accuracy advantage).
- Assignments filtered at combined confidence 0.4–0.5 should maximize the F1/2-score relative to looser or stricter cutoffs, per the authors' benchmarks.
- Verify MAPseq version is 1.2.6+ so -otutable contains counts from all samples rather than only the first.

## Limitations

- MAPseq v1.2.5's '-otutable' option used only counts from the first sample; fixed in v1.2.6, so older versions produce incorrect multi-sample tables.
- The combined confidence score depends on preoptimized per-level identity cutoffs; recommended cutoff (0.4 or 0.5) is benchmark-specific and moving the binary alone breaks data-file lookup.
- Low-quality reference sequences or long queries (>1200 bp) caused classification issues in earlier versions; LTP taxonomy was dropped due to low coverage.

## Evidence

- [results] On artificial mock community data, MAPseq recovers expected abundances better than other tools, at the species, genus and family levels.: [evidence span withheld — anchor status: not_found]
- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] mapseq -otutable <sample1.mseq> <sample2.mseq> ...  Generates a tab separated value (tsv) file with the counts for each sample: [evidence span withheld — anchor status: external_doc]
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [readme] 1.2.6 (24 Mar 2020) - Fixed '-otutable' option using only counts from first sample.: "1.2.6 (24 Mar 2020)
- Fixed "-otutable" option using only counts from first sample."
