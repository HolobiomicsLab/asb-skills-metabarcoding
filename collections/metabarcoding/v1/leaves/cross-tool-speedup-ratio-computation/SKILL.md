---
name: cross-tool-speedup-ratio-computation
description: Use when you need to quantify how much faster (and more memory-frugal)
  one sequence-analysis tool is than baselines on the same input, e.g. validating
  MAPseq's ~10x vs NINJA-OPS and ~100x vs VSEARCH speedups.
when_to_use_negative:
- Input is shotgun metagenomic data rather than amplicon rRNA reads — the reported
  10x/100x ratios are specific to amplicon benchmarks (Fig. 1a)
- You need accuracy (F½-score) comparisons rather than runtime/memory benchmarking
- The baseline tools cannot be run on the identical input under the same profiling
  wrapper, making ratios non-comparable
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3316
tools:
- name: MAPseq
  role: Tool under benchmark; run on amplicon data with profiling wrapper to record
    runtime and peak memory
  repo: https://github.com/jfmrod/mapseq
- name: NINJA-OPS
  role: Closest-competitor baseline tool run on identical amplicon input to compute
    the ~10x speedup ratio
- name: VSEARCH
  role: Baseline tool run on identical amplicon input to compute the ~100x speedup
    ratio
- name: autoconf
  role: Generates the './configure' script used to configure MAPseq before building
- name: GNU make
  role: Compiles MAPseq ('make', optionally 'make check' for package self-tests)
- name: gcc
  role: C compiler invoked by configure/make to build MAPseq from source
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:898a545343419a4c05650fa5de8d8542111df006f728c580bc77f4c6e0d03a79
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - NINJA-OPS
  - VSEARCH
  - autoconf
  - gcc
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Cross-tool speedup ratio computation

## Summary

Benchmark MAPseq against competing rRNA analysis tools (NINJA-OPS, VSEARCH, USEARCH) on identical amplicon input, recording wall-clock runtime and peak memory to compute order-of-magnitude speedup ratios that reproduce the paper's Fig. 1a-b claims.

## When to use

Use when you need to quantify how much faster (and more memory-frugal) one sequence-analysis tool is than baselines on the same input, e.g. validating MAPseq's ~10x vs NINJA-OPS and ~100x vs VSEARCH speedups.

## When NOT to use

- Input is shotgun metagenomic data rather than amplicon rRNA reads — the reported 10x/100x ratios are specific to amplicon benchmarks (Fig. 1a)
- You need accuracy (F½-score) comparisons rather than runtime/memory benchmarking
- The baseline tools cannot be run on the identical input under the same profiling wrapper, making ratios non-comparable

## Inputs

- Amplicon rRNA benchmark dataset (demultiplexed, quality-filtered FASTA, e.g. rawseqs.fa)
- MAPseq v1.2.6 source from github.com/jfmrod/mapseq or a binary release tarball
- Baseline tool binaries: NINJA-OPS, VSEARCH, USEARCH

## Outputs

- Per-tool wall-clock runtime (seconds) and peak resident memory (MB) measurements
- Computed speedup ratios of MAPseq vs NINJA-OPS, VSEARCH and USEARCH
- Summary table benchmark_runtime_memory.csv

## How to apply

Build MAPseq v1.2.6 from source (github.com/jfmrod/mapseq) with ./configure followed by GNU make, optionally running 'make check' for self-tests, or unpack the binary release tarball. Load the amplicon rRNA benchmark dataset (demultiplexed, quality-filtered FASTA) and run MAPseq over it under a profiling wrapper such as /usr/bin/time -v to capture wall-clock seconds and peak resident memory (MB). Run NINJA-OPS, VSEARCH and USEARCH on the identical amplicon input under the same wrapper so measurements are comparable. Divide each baseline's runtime by MAPseq's to get speedup ratios, and compare memory footprints; accept the result if the summary table shows roughly 10x over NINJA-OPS and 100x over VSEARCH (Fig. 1a-b) and MAPseq has the lowest memory footprint. Write per-tool runtime and peak-memory values plus ratios to benchmark_runtime_memory.csv.

## Related tools

- **MAPseq** (Tool under benchmark; run on amplicon data with profiling wrapper to record runtime and peak memory) — https://github.com/jfmrod/mapseq
- **NINJA-OPS** (Closest-competitor baseline tool run on identical amplicon input to compute the ~10x speedup ratio)
- **VSEARCH** (Baseline tool run on identical amplicon input to compute the ~100x speedup ratio)
- **autoconf** (Generates the './configure' script used to configure MAPseq before building)
- **GNU make** (Compiles MAPseq ('make', optionally 'make check' for package self-tests))
- **gcc** (C compiler invoked by configure/make to build MAPseq from source)

## Examples

```
git clone https://github.com/jfmrod/MAPseq.git && cd MAPseq && ./bootstrap && ./configure && make && make check && /usr/bin/time -v mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- benchmark_runtime_memory.csv contains runtime and peak-memory entries for MAPseq, NINJA-OPS and VSEARCH
- Computed speedup reproduces the reported order-of-magnitude ratios: roughly 10x over NINJA-OPS and 100x over VSEARCH (Fig. 1a-b)
- MAPseq shows the lowest memory footprint among the compared tools
- All tools were run on the identical amplicon input under the same profiling wrapper (/usr/bin/time -v)
- MAPseq build passes 'make check' package self-tests before benchmarking

## Limitations

- The 10x/100x speedups were measured on amplicon data; relative performance on shotgun data may differ
- Building from a cloned git repository requires extra dependencies (svn, autotools/autoconf, wget, git, libncurses5-dev, libtool) and a ./bootstrap step
- Moving only the mapseq binary elsewhere breaks the installation because data files are searched relative to the binary's path
- For a cloned repository, ./bootstrap is needed before ./configure, and ./setup.sh downloads eutils and reference data files

## Evidence

- [results] On amplicon data, MAPseq is ten times faster than its closest competitor NINJA-OPS and a hundred times faster than VSEARCH (Fig. 1a).: [evidence span withheld — anchor status: not_found]
- [readme] MAPseq v1.2.6 (24 Mar 2020): "MAPseq v1.2.6 (24 Mar 2020)"
- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] ./bootstrap     # this step is only needed if you cloned the repository, you will also need to install autotools/autoconf packages: [evidence span withheld — anchor status: external_doc]
- [readme] Moving only the binary elsewhere will break the installation though, as the data files are searched for in relation to the binary's path.: [evidence span withheld — anchor status: external_doc]
