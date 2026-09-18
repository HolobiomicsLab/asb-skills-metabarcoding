---
name: peak-memory-footprint-measurement
description: Use when comparing wall-clock runtime and peak resident memory (e.g.
  via a profiling wrapper such as /usr/bin/time -v) of rRNA classification tools like
  MAPseq, NINJA-OPS and VSEARCH on identical amplicon inputs.
when_to_use_negative:
- Input is shotgun metagenomic data rather than amplicon rRNA reads, if the goal is
  to reproduce the amplicon-specific speedup figures of Fig. 1a-b
- Input is already a computed runtime/memory table needing only analysis rather than
  fresh benchmarking
- The comparison targets accuracy (F½ score) rather than runtime and memory footprint
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MAPseq
  role: Tool under benchmark; run on the amplicon dataset while profiling wall-clock
    time and peak resident memory
  repo: https://github.com/jfmrod/mapseq
- name: NINJA-OPS
  role: Closest-competitor baseline for runtime and memory comparison on identical
    amplicon input
- name: VSEARCH
  role: Baseline tool for runtime and memory comparison on identical amplicon input
- name: autoconf
  role: Generates the ./configure script used to build MAPseq from source before benchmarking
- name: GNU make
  role: Compiles MAPseq ('make', optionally 'make check' for self-tests) prior to
    benchmarking
- name: gcc
  role: Compiler used when building MAPseq via ./configure CC=... and make
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:c392dd704018e95fd2f6718d053f03261dba0b8c359462798da1ceb6cb5832d2
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3
n_claims: 10
---

# peak-memory-footprint-measurement

## Summary

A benchmarking skill for measuring wall-clock runtime and peak resident memory of rRNA sequence classification tools on amplicon data, used to quantify MAPseq's speedup (up to 100x) and lowest memory footprint relative to NINJA-OPS and VSEARCH.

## When to use

Use when comparing wall-clock runtime and peak resident memory (e.g. via a profiling wrapper such as /usr/bin/time -v) of rRNA classification tools like MAPseq, NINJA-OPS and VSEARCH on identical amplicon inputs.

## When NOT to use

- Input is shotgun metagenomic data rather than amplicon rRNA reads, if the goal is to reproduce the amplicon-specific speedup figures of Fig. 1a-b
- Input is already a computed runtime/memory table needing only analysis rather than fresh benchmarking
- The comparison targets accuracy (F½ score) rather than runtime and memory footprint

## Inputs

- Amplicon rRNA benchmark dataset in FASTA form (demultiplexed, quality-filtered sequences, e.g. rawseqs.fa)
- Executable builds of MAPseq v1.2.6, NINJA-OPS and VSEARCH
- Profiling wrapper such as /usr/bin/time -v

## Outputs

- benchmark_runtime_memory.csv containing per-tool runtime (seconds) and peak memory (MB) for MAPseq, NINJA-OPS and VSEARCH plus computed speedup ratios
- Per-run profiling logs with wall-clock time and peak resident memory

## How to apply

Build MAPseq v1.2.6 from source (git clone https://github.com/jfmrod/MAPseq.git; run ./setup.sh, ./bootstrap, ./configure, make, make install) so the tool under test matches the published benchmark version. Run MAPseq over the amplicon rRNA benchmark dataset, wrapping the process with a profiling tool (e.g. /usr/bin/time -v) to capture wall-clock seconds and peak resident set size in MB. Run the baselines NINJA-OPS and VSEARCH on the identical amplicon input under the same profiling wrapper so measurements are comparable. Compute speedup ratios (expect roughly 10x over NINJA-OPS and 100x over VSEARCH per Fig. 1a-b) and memory-footprint differences, confirming MAPseq has the lowest memory footprint. Assemble per-tool runtime (seconds) and peak memory (MB) plus computed ratios into benchmark_runtime_memory.csv.

## Related tools

- **MAPseq** (Tool under benchmark; run on the amplicon dataset while profiling wall-clock time and peak resident memory) — https://github.com/jfmrod/mapseq
- **NINJA-OPS** (Closest-competitor baseline for runtime and memory comparison on identical amplicon input)
- **VSEARCH** (Baseline tool for runtime and memory comparison on identical amplicon input)
- **autoconf** (Generates the ./configure script used to build MAPseq from source before benchmarking)
- **GNU make** (Compiles MAPseq ('make', optionally 'make check' for self-tests) prior to benchmarking)
- **gcc** (Compiler used when building MAPseq via ./configure CC=... and make)

## Examples

```
/usr/bin/time -v mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- benchmark_runtime_memory.csv contains runtime and peak-memory entries for all three tools: MAPseq, NINJA-OPS and VSEARCH
- Computed speedups reproduce the reported order-of-magnitude ratios: roughly 10x over NINJA-OPS and 100x over VSEARCH (per Fig. 1a-b)
- MAPseq shows the lowest peak resident memory (MB) among the compared tools
- All tools were profiled with the same wrapper (e.g. /usr/bin/time -v) on the identical amplicon input file
- MAPseq version string matches the benchmark version, MAPseq v1.2.6 (24 Mar 2020)

## Limitations

- The 100x speedup claim is 'up to' and applies to amplicon data comparisons in the article's benchmarks; other datasets may show smaller ratios
- MAPseq binary installs must keep data files co-located with the binary: 'Moving only the binary elsewhere will break the installation though, as the data files are searched for in relation to the binary's path'
- Version-specific fixes (e.g. 1.2.2 multithreaded race condition, 1.2.6 -otutable fix) mean benchmarks from older MAPseq versions may not reproduce

## Evidence

- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [abstract] Source code and binaries freely available: "Source code and binaries are freely available at https://github.com/jfmrod/mapseq."
- [readme] README install prerequisites: [evidence span withheld — anchor status: external_doc]
- [readme] README bootstrap step: [evidence span withheld — anchor status: external_doc]
- [readme] README default usage: [evidence span withheld — anchor status: external_doc]
