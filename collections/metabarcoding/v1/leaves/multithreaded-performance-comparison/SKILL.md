---
name: multithreaded-performance-comparison
description: Use when a tool release claims speed or memory improvements and you need
  to quantify speedup factors, memory ratios, and output equivalence across single-core
  and multi-core runs on identical input data.
when_to_use_negative:
- The input FASTA is not strictly dereplicated — swarm 3.0 makes strict dereplication
  mandatory, so raw or under-dereplicated reads will break the benchmark
- The scientific question is about clustering quality or biological OTU composition
  rather than runtime and memory performance
- You intend to use large d values (d >= 2 or 3) or the fastidious option, since the
  benchmark targets default d = 1 behaviour and fastidious can significantly increase
  memory footprint
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3293
tools:
- name: swarm v3
  role: Newer version under test; clustered with default d = 1 under 1 and 16 threads
    to measure speedup and memory reduction
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline version providing reference runtime, memory footprint, and seed FASTA
    output for equivalence checks
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:6626577f152b0e712f6f3139f929f21f622bb511fd22e55e24cdfa87b6aab4ce
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v2
  - swarm v3
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# multithreaded-performance-comparison

## Summary

Benchmark a new tool version against its predecessor by measuring wall-clock runtime and peak resident memory under fixed thread counts (e.g. 1 vs 16 cores), while verifying that outputs remain identical. Applied to swarm v3 vs v2 on large dereplicated SSU-rRNA amplicon FASTA datasets.

## When to use

Use when a tool release claims speed or memory improvements and you need to quantify speedup factors, memory ratios, and output equivalence across single-core and multi-core runs on identical input data.

## When NOT to use

- The input FASTA is not strictly dereplicated — swarm 3.0 makes strict dereplication mandatory, so raw or under-dereplicated reads will break the benchmark
- The scientific question is about clustering quality or biological OTU composition rather than runtime and memory performance
- You intend to use large d values (d >= 2 or 3) or the fastidious option, since the benchmark targets default d = 1 behaviour and fastidious can significantly increase memory footprint

## Inputs

- Dereplicated amplicon FASTA files (e.g. SSU-rRNA V4: 10.6M unique sequences, 380 bp average; V9: 10.6M unique sequences, 130 bp average) with occurrence counts in headers
- Binaries of the two tool versions being compared (swarm v2 and swarm v3)
- External timing/profiling utility for wall-clock runtime and peak resident memory

## Outputs

- benchmark_results.csv with columns: dataset, version, threads, runtime, peak memory, speedup, memory ratio
- Cluster representatives (seeds) FASTA per run, produced via -w/--seeds, for byte-identical comparison

## How to apply

Obtain both tool versions (e.g. swarm v2 and swarm v3 binaries from github.com/torognes/swarm) and a dereplicated FASTA dataset (e.g. 10.6 million unique SSU-rRNA V4 sequences of ~380 bp, and a V9 dataset of ~130 bp reads). Run each version with default parameters (local threshold d = 1) under --threads 1 and --threads 16, wrapping each run with an external timing utility to record wall-clock runtime and peak resident memory. Write cluster representatives (seeds) with the -w/--seeds option so clustering results can be compared. Compute the speedup factor (v2 runtime / v3 runtime) and memory ratio (v3 memory / v2 memory) per dataset and thread count, and validate that the seed FASTA from the new version is byte-identical to the reference. Rationale: swarm v3's optimized CPU usage and multithreading should give >7x speedup on 1 core, ~10x on 16 cores, and ~50% memory reduction without altering d = 1 clustering results.

## Related tools

- **swarm v3** (Newer version under test; clustered with default d = 1 under 1 and 16 threads to measure speedup and memory reduction) — https://github.com/torognes/swarm
- **swarm v2** (Baseline version providing reference runtime, memory footprint, and seed FASTA output for equivalence checks) — https://github.com/torognes/swarm

## Examples

```
./swarm --threads 16 --seeds v3_v4_seeds.fasta v4_amplicons.fasta
```

## Evaluation signals

- V4 dataset speedup exceeds 7x on 1 core and is approximately 10x on 16 cores
- V4 memory ratio (v3/v2) falls between 0.4 and 0.6 (i.e. about half of v2's memory)
- The v3 V4 seed FASTA (written with -w/--seeds) is byte-identical to the v2 reference output
- Each version x thread-count combination is run on both V4 and V9 datasets under identical conditions (8 runs total)
- Wall-clock time and peak resident memory come from an external timing utility, not from tool self-reporting

## Limitations

- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so speedup claims may not generalize to non-default thresholds
- The fastidious option can significantly increase memory footprint, confounding memory benchmarks if enabled
- swarm's memory footprint is roughly 0.6 times the input FASTA size even in the baseline case, so absolute numbers scale with dataset size

## Evidence

- [abstract] optimized CPU-usage and multithreading (more than 7 times faster with default parameters): "optimized CPU-usage and multithreading (more than 7 times faster with default parameters)"
- [abstract] reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [methods] ./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta: "./swarm \
    --fastidious \
    --threads 4 \
    --seeds cluster_representatives.fasta \
    amplicons.fasta"
- [results] A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset: "A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset"
- [readme] The memory footprint of swarm is roughly 0.6 times the size of the input fasta file.: "The memory footprint of **swarm** is roughly 0.6 times the size of the
input fasta file. When using the fastidious option, memory footprint
can increase significantly."
