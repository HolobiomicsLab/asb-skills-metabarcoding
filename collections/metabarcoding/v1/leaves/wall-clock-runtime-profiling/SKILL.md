---
name: wall-clock-runtime-profiling
description: Use when comparing the computational performance of sequence-analysis
  tools on identical inputs, e.g. verifying claims like 'up to one hundred times faster'
  with runtime and memory measurements.
when_to_use_negative:
- The goal is accuracy evaluation (F½ score) rather than runtime/memory performance
  comparison
- Input is shotgun metagenomic data when the benchmark targets amplicon data specifically
- Tools cannot be run on identical inputs or hardware, making speedup ratios meaningless
edam_operation: http://edamontology.org/operation_0334
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0091
tools:
- name: MAPseq
  role: Tool under benchmark; run over the amplicon dataset to record runtime and
    peak memory
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Generates the './configure' script used to build MAPseq v1.2.6 before benchmarking
- name: GNU make
  role: Compiles MAPseq ('make', optionally 'make check' for self-tests) prior to
    profiling
- name: gcc
  role: C compiler invoked through the configure/make build of MAPseq
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:b770e38ea085926881df8bf7f697f0a364c292c815cc8f514df9c1ee7200d9f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  - gcc
  edam_operation: http://edamontology.org/operation_0334
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# wall-clock-runtime-profiling

## Summary

Benchmark rRNA analysis tools by measuring wall-clock runtime and peak resident memory under a profiling wrapper, then compute relative speedup ratios. Used to reproduce MAPseq's reported order-of-magnitude speedups over NINJA-OPS and VSEARCH on amplicon data.

## When to use

Use when comparing the computational performance of sequence-analysis tools on identical inputs, e.g. verifying claims like 'up to one hundred times faster' with runtime and memory measurements.

## When NOT to use

- The goal is accuracy evaluation (F½ score) rather than runtime/memory performance comparison
- Input is shotgun metagenomic data when the benchmark targets amplicon data specifically
- Tools cannot be run on identical inputs or hardware, making speedup ratios meaningless

## Inputs

- amplicon rRNA benchmark dataset (demultiplexed, quality-filtered FASTA, e.g. rawseqs.fa)
- built executables of MAPseq v1.2.6, NINJA-OPS, and VSEARCH
- profiling wrapper (e.g. /usr/bin/time -v)

## Outputs

- per-tool wall-clock runtime (seconds) and peak resident memory (MB) measurements
- computed speedup ratios and memory-footprint differences
- summary table (e.g. benchmark_runtime_memory.csv)

## How to apply

Build each tool from source under identical conditions (for MAPseq v1.2.6: './setup.sh', './bootstrap', './configure', 'make'), then run every tool over the same amplicon rRNA benchmark input wrapped in a resource profiler such as '/usr/bin/time -v' to capture wall-clock seconds and peak resident memory. Keep thread counts and hardware constant across tools so ratios reflect the software, not configuration differences. Compute per-tool speedup ratios and memory-footprint differences; expect MAPseq to be roughly 10x faster than NINJA-OPS and 100x faster than VSEARCH on amplicon data (Fig. 1a-b), with the lowest memory footprint. Assemble the measurements and ratios into a summary table (e.g. benchmark_runtime_memory.csv) for validation against the published order-of-magnitude claims.

## Related tools

- **MAPseq** (Tool under benchmark; run over the amplicon dataset to record runtime and peak memory) — https://github.com/jfmrod/mapseq
- **autoconf** (Generates the './configure' script used to build MAPseq v1.2.6 before benchmarking)
- **GNU make** (Compiles MAPseq ('make', optionally 'make check' for self-tests) prior to profiling)
- **gcc** (C compiler invoked through the configure/make build of MAPseq)

## Examples

```
/usr/bin/time -v mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- Summary table contains runtime and peak-memory entries for all three tools (MAPseq, NINJA-OPS, VSEARCH)
- Reproduced speedups are of the reported order of magnitude: ~10x over NINJA-OPS and ~100x over VSEARCH on amplicon data (Fig. 1a-b)
- MAPseq shows the lowest memory footprint among the compared tools
- All tools were profiled on the identical amplicon input under the same wrapper (/usr/bin/time -v) and hardware
- MAPseq build passes its package self-tests ('make check') before measurements are taken

## Limitations

- Reported speedups are order-of-magnitude claims ('up to one hundred times faster'); exact ratios depend on hardware, threads, and dataset
- The README does not document the amplicon benchmark dataset itself; it must be sourced separately
- Runtime/memory comparisons on data other than amplicon rRNA may not reproduce the published ratios

## Evidence

- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] build and self-test workflow: "./setup.sh      # downloads eutils and the reference data files
./bootstrap     # this step is only needed if you cloned the repository, you will also need to install autotools/autoconf"
- [intro] MAPseq version under benchmark: "MAPseq v1.2.6 (24 Mar 2020)"
- [readme] MAPseq default invocation on FASTA input: [evidence span withheld — anchor status: external_doc]
