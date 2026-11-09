---
name: f-half-score-classifier-benchmarking
description: Use when comparing rRNA amplicon classifiers on reads of known identity,
  to reproduce MAPseq-style F½-score rankings against genus-level and 98%-identity
  hOTU placement targets.
when_to_use_negative:
- Input reads are environmental sequences without known-identity truth labels — F½
  benchmarking requires trusted genus/hOTU placements
- The goal is performance/throughput benchmarking (runtime, memory) rather than classification
  accuracy; the F½ metric alone does not capture the 'one hundred times faster' speed
  dimension
- Input is already a feature/count table (e.g. mapseq -otutable output); classification
  and scoring happen on raw reads
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MAPseq
  role: Classifier under test; assigns taxonomy and hierarchical OTU mappings to amplicon
    rRNA reads with confidence estimates; its F½ maxima (~0.86 genus, ~0.96 hOTU)
    define the benchmark ceiling
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Used to create the 'configure' script when building MAPseq v1.2.6 from source
    for the benchmark
  repo: https://github.com/jfmrod/mapseq
- name: GNU make
  role: Compiles and installs MAPseq (make, make install) prior to running the benchmark
  repo: https://github.com/jfmrod/mapseq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:b13c141062ec5a621f2d90433896ffb85b9968f2abf655dcec063409545510f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# f-half-score-classifier-benchmarking

## Summary

Benchmark amplicon rRNA sequence classifiers by computing F½ scores of taxonomy/genus assignments and hierarchical OTU mappings against reads of known identity, as done in the MAPseq evaluation where MAPseq reached F½ ≈ 0.86 (genus) and ≈ 0.96 (98%-identity hOTUs).

## When to use

Use when comparing rRNA amplicon classifiers on reads of known identity, to reproduce MAPseq-style F½-score rankings against genus-level and 98%-identity hOTU placement targets.

## When NOT to use

- Input reads are environmental sequences without known-identity truth labels — F½ benchmarking requires trusted genus/hOTU placements
- The goal is performance/throughput benchmarking (runtime, memory) rather than classification accuracy; the F½ metric alone does not capture the 'one hundred times faster' speed dimension
- Input is already a feature/count table (e.g. mapseq -otutable output); classification and scoring happen on raw reads

## Inputs

- Benchmark amplicon rRNA reads of known identity (FASTA, previously demultiplexed and quality filtered)
- Reference full-length rRNA sequence databases with known taxonomies and precomputed high-quality OTU clusters (MAPseq standard reference dataset)
- Taxonomy truth labels per read (genus-level categories and hOTUs clustered at 98% sequence identity)

## Outputs

- Per-read taxonomy and OTU classifications with confidence values (MAPseq .mseq format)
- fhalf_scores.csv — a table of per-tool F½ scores for genus-level and 98%-identity hOTU benchmarks

## How to apply

Build MAPseq v1.2.6 from https://github.com/jfmrod/mapseq with the GNU autotools toolchain (./bootstrap, ./configure, make, make install). Run MAPseq on benchmark amplicon reads of known identity against the reference rRNA databases to obtain taxonomy classifications and hierarchical OTU mappings, then run the baseline classification tools on the same reads. Compare each tool's genus-level assignments to the known-identity truth labels and compute F½ scores (expecting a maximum of ~0.86 at 500 bp with MAPseq up to 30% better than existing solutions). Separately compare assignments against hOTUs clustered at 98% sequence identity and compute F½ scores (expecting a maximum of ~0.96). Assemble per-tool F½ scores for both benchmark types into fhalf_scores.csv and verify the table reproduces the relative ranking of Fig. 1c-f.

## Related tools

- **MAPseq** (Classifier under test; assigns taxonomy and hierarchical OTU mappings to amplicon rRNA reads with confidence estimates; its F½ maxima (~0.86 genus, ~0.96 hOTU) define the benchmark ceiling) — https://github.com/jfmrod/mapseq
- **autoconf** (Used to create the 'configure' script when building MAPseq v1.2.6 from source for the benchmark) — https://github.com/jfmrod/mapseq
- **GNU make** (Compiles and installs MAPseq (make, make install) prior to running the benchmark) — https://github.com/jfmrod/mapseq

## Examples

```
git clone https://github.com/jfmrod/MAPseq.git && cd MAPseq && ./setup.sh && ./bootstrap && ./configure && make && make install && mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- MAPseq attains the reported maxima: F½ ≈ 0.86 for genus-level categories and F½ ≈ 0.96 for 98%-identity hOTUs
- The assembled fhalf_scores.csv reproduces the relative ranking of Fig. 1c-f
- MAPseq's genus-mapping F½ exceeds baseline tools' scores by up to 30%
- MAPseq is run at a combined confidence cutoff of 0.4–0.5, the value reported to yield the highest F½-score in the authors' benchmarks

## Limitations

- Confidence cutoff choice affects accuracy: the README recommends a combined confidence cutoff of 0.4 or 0.5 because it yielded the highest F½-score; other cutoffs may underperform
- Taxonomy assignments can be reduced by uncertain annotation — v1.2.4 added -ignoreEmptyTax to prevent second hits with missing taxonomic labels from lowering confidence of the top hit, but it is off by default until thorough benchmarks are performed
- F½ scores are computed on reads of known identity; accuracy against genus and 98%-identity hOTU targets is bounded by reference database coverage and quality

## Evidence

- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] MAPseq v1.2.6 (24 Mar 2020): "MAPseq v1.2.6 (24 Mar 2020)"
- [readme] confidence cutoff of 0.4, or 0.5: [evidence span withheld — anchor status: external_doc]
- [readme] build-from-source steps: [evidence span withheld — anchor status: external_doc]
- [full_text] hOTU clustering at 98% identity: [evidence span withheld — anchor status: not_found]
