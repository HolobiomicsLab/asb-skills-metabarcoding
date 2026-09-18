---
name: peak-memory-footprint-profiling
description: Use when comparing performance of two tool versions or configurations
  (runtime speedup, memory-footprint ratio) on amplicon clustering inputs, or when
  validating that a faster implementation reproduces prior clustering results exactly.
when_to_use_negative:
- The input is a filtered occurrence table or other post-clustering output — this
  skill benchmarks clustering runs, not downstream tables.
- The task is clustering quality assessment rather than performance profiling (runtime/memory/identical-output
  validation).
- The clustering threshold d >= 2 is required — the >7x speedup and ~50% memory-reduction
  claims apply to the default d = 1 fast algorithm; a 15% slowdown was observed for
  d >= 2 in certain conditions (18SV9 dataset).
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: swarm v3
  role: Tool under test; benchmarked for runtime speedup (>7x on 1 core, ~10x on 16
    cores) and memory footprint (~50% reduction) at default d = 1
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline version providing reference runtime, peak-memory, and seed-output
    (OTU) measurements against which v3 is compared
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:4cbce232dba9ddc20697601b97ba71d324011e78b4f46100790f4fb8ce31734e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v2
  - swarm v3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# peak-memory-footprint-profiling

## Summary

A benchmarking skill for measuring wall-clock runtime and peak resident memory of a command-line bioinformatics tool (here swarm v3 vs swarm v2) across thread counts on large dereplicated amplicon FASTA datasets, while verifying that outputs remain identical.

## When to use

Use when comparing performance of two tool versions or configurations (runtime speedup, memory-footprint ratio) on amplicon clustering inputs, or when validating that a faster implementation reproduces prior clustering results exactly.

## When NOT to use

- The input is a filtered occurrence table or other post-clustering output — this skill benchmarks clustering runs, not downstream tables.
- The task is clustering quality assessment rather than performance profiling (runtime/memory/identical-output validation).
- The clustering threshold d >= 2 is required — the >7x speedup and ~50% memory-reduction claims apply to the default d = 1 fast algorithm; a 15% slowdown was observed for d >= 2 in certain conditions (18SV9 dataset).

## Inputs

- Dereplicated amplicon FASTA dataset (V4: 10.6M unique SSU-rRNA sequences, ~380 bp) with abundance-bearing headers
- Dereplicated amplicon FASTA dataset (V9: 10.6M unique SSU-rRNA sequences, ~130 bp) with abundance-bearing headers
- swarm v2 and swarm v3 binaries

## Outputs

- benchmark_results.csv with columns (dataset, version, threads, runtime, peak memory, speedup, memory ratio)
- Cluster representatives (seeds) FASTA per run via -w/--seeds, used for byte-identical v3-vs-v2 comparison

## How to apply

Obtain both tool binaries (swarm v2 and swarm v3 from github.com/torognes/swarm) and dereplicated amplicon FASTA inputs (V4: 10.6M unique sequences, 380 bp average; V9: 10.6M unique sequences, 130 bp average). Run each version under default parameters (local threshold d = 1) with an external timing utility, once with --threads 1 and once with --threads 16, writing cluster representatives (seeds) with the -w/--seeds option so clustering output can be compared. Assemble measurements into a benchmark_results.csv (dataset, version, threads, runtime, peak memory, speedup, memory ratio). Validate that the V4 speedup exceeds 7x on 1 core and is approximately 10x on 16 cores, that the V4 memory ratio (v3/v2) falls between 0.4 and 0.6, and that the v3 OTU/seed output is byte-identical to the v2 reference. Note that swarm 3.0 makes strict dereplication of input sequences mandatory, so inputs must already carry abundance counts (e.g. _1000 suffixes).

## Related tools

- **swarm v3** (Tool under test; benchmarked for runtime speedup (>7x on 1 core, ~10x on 16 cores) and memory footprint (~50% reduction) at default d = 1) — https://github.com/torognes/swarm
- **swarm v2** (Baseline version providing reference runtime, peak-memory, and seed-output (OTU) measurements against which v3 is compared) — https://github.com/torognes/swarm

## Examples

```
/usr/bin/time -v ./swarm --threads 16 -w seeds_v3.fasta amplicons_V4.fasta
```

## Evaluation signals

- V4 speedup factor exceeds 7x on 1 core and is approximately 10x on 16 cores
- V4 memory-footprint ratio (v3/v2) falls between 0.4 and 0.6
- v3 V4 seed/OTU output file matches the v2 reference output exactly (byte-identical clustering results)
- benchmark_results.csv contains runtime and peak-memory rows for all 2 versions x 2 thread counts x 2 datasets (V4 and V9)
- All runs use default parameters (d = 1) on properly dereplicated FASTA input with occurrence counts in headers

## Limitations

- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so performance claims are tied to the default d = 1.
- Memory footprint of swarm is roughly 0.6 times the size of the input FASTA file, and the fastidious option can increase memory consumption significantly (controllable via -c and -y).
- swarm 3.0 makes strict dereplication mandatory; non-dereplicated input or headers lacking occurrence counts will not work properly.

## Evidence

- [abstract] swarm v3 is more than 7 times faster with default parameters due to optimized CPU-usage and multithreading: "optimized CPU-usage and multithreading (more than 7 times faster with default parameters)"
- [abstract] swarm v3 reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [readme] Memory footprint scales with input FASTA size: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
- [methods] Benchmark command pattern with threads and seeds output: "./swarm \
    --fastidious \
    --threads 4 \
    --seeds cluster_representatives.fasta \
    amplicons.fasta"
- [methods] Slowdown for d >= 2: "A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset"
- [readme] Dereplication is mandatory in v3: "strict dereplication of input sequences is now **mandatory**,"
