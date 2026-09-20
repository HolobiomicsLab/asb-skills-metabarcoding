---
name: hotu-clustering-read-mapping
description: Use when benchmarking rRNA amplicon classification tools on reads of
  known identity, comparing genus-level assignments or 98%-identity hOTU mappings
  via F½ scores.
when_to_use_negative:
- The input is shotgun metagenomic data requiring functional profiling rather than
  rRNA amplicon classification benchmarks
- You need novel OTU clustering from raw reads rather than placement of reads onto
  pre-generated hOTU clusters at 98% identity
- Reads lack known-identity truth labels, so F½-score benchmarking against genus or
  hOTU targets cannot be computed
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: MAPseq
  role: Primary classification tool that assigns taxonomy and hierarchical OTU classifications
    to rRNA reads using a k-mer search with confidence estimates; benchmarked against
    baseline tools with F½ scores
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Generates the 'configure' script used to build MAPseq from source before running
    the benchmark
  repo: https://github.com/jfmrod/mapseq
- name: GNU make
  role: Compiles and installs MAPseq (make, make install) after configuration
  repo: https://github.com/jfmrod/mapseq
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:8ada01b726d480953df9ab7754d543be647e00695f6865067db22c4589ee5ce6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# hotu-clustering-read-mapping

## Summary

Benchmarking skill for evaluating rRNA amplicon read placement against hierarchical OTUs (hOTUs clustered at 98% sequence identity) and genus-level taxonomic categories, using MAPseq's F½-score comparisons against other classification tools.

## When to use

Use when benchmarking rRNA amplicon classification tools on reads of known identity, comparing genus-level assignments or 98%-identity hOTU mappings via F½ scores.

## When NOT to use

- The input is shotgun metagenomic data requiring functional profiling rather than rRNA amplicon classification benchmarks
- You need novel OTU clustering from raw reads rather than placement of reads onto pre-generated hOTU clusters at 98% identity
- Reads lack known-identity truth labels, so F½-score benchmarking against genus or hOTU targets cannot be computed

## Inputs

- Benchmark amplicon rRNA reads of known identity (FASTA, demultiplexed and quality filtered)
- Reference full-length rRNA sequence database with known taxonomies and pre-generated high-quality OTU clusters (MAPseq mapref)
- hOTU reference clusters at 98% sequence identity and genus-level taxonomic category truth labels

## Outputs

- fhalf_scores.csv: per-tool F½ scores for genus mapping and 98%-identity hOTU mapping
- MAPseq .mseq classification files with taxonomy and OTU assignments plus confidence values

## How to apply

Obtain MAPseq v1.2.6 from GitHub and build it with the GNU autotools toolchain (./bootstrap, ./configure, make, make install; on Ubuntu install build-essential, wget, subversion, git, libncurses5-dev, libtool first). Run MAPseq on the benchmark amplicon reads of known identity against the reference rRNA databases to produce taxonomy classifications and hierarchical OTU mappings, and run the baseline classification tools on the same reads. Compare each tool's genus-level assignments to the known-identity truth labels and compute F½ scores, expecting a maximum of ~0.86 at 500 bp with MAPseq up to 30% better than existing solutions. Compare each tool's assignments against hOTUs clustered at 98% sequence identity and compute F½ scores, expecting a maximum of ~0.96. Assemble per-tool F½ scores for both benchmark types into fhalf_scores.csv and validate the relative ranking against Fig. 1c-f; the README recommends a combined confidence cutoff of 0.4 or 0.5, which yielded the highest F1/2-score in their benchmarks.

## Related tools

- **MAPseq** (Primary classification tool that assigns taxonomy and hierarchical OTU classifications to rRNA reads using a k-mer search with confidence estimates; benchmarked against baseline tools with F½ scores) — https://github.com/jfmrod/mapseq
- **autoconf** (Generates the 'configure' script used to build MAPseq from source before running the benchmark) — https://github.com/jfmrod/mapseq
- **GNU make** (Compiles and installs MAPseq (make, make install) after configuration) — https://github.com/jfmrod/mapseq

## Examples

```
git clone https://github.com/jfmrod/MAPseq.git && cd MAPseq && ./bootstrap && ./configure && make && sudo make install && mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- fhalf_scores.csv reproduces the relative ranking of Fig. 1c-f
- MAPseq attains maximum F½ ≈ 0.86 for genus-level categories
- MAPseq attains maximum F½ ≈ 0.96 for 98%-identity hOTUs
- MAPseq exceeds baseline tools' F½ by up to 30%
- Combined confidence cutoff of 0.4–0.5 is used, matching the value that yielded the highest F1/2-score in the authors' benchmarks

## Limitations

- Benchmark maxima are reported for 500 bp reads; performance at other read lengths is not guaranteed by this skill
- MAPseq v1.2.4's -ignoreEmptyTax option is off by default until thorough benchmarks are performed, and second hits with missing taxonomic labels can decrease confidence of the top hit
- MAPseq dropped LTP taxonomy due to low coverage in v1.2.1, limiting available taxonomic references

## Evidence

- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] confidence cutoff recommendation: [evidence span withheld — anchor status: external_doc]
- [intro] MAPseq version: "MAPseq v1.2.6 (24 Mar 2020)"
- [abstract] hierarchical OTU mapping in a single run: "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences in both amplicon and shotgun sequencing strategies"
