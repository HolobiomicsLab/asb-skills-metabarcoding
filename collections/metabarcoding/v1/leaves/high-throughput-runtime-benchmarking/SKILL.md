---
name: high-throughput-runtime-benchmarking
description: Use when comparing runtime speed and memory footprint of two versions
  of a clustering/bioinformatics tool on very large (10.6M-sequence) FASTA inputs,
  and when output equivalence must be verified between versions.
when_to_use_negative:
- Input is a raw FASTQ or non-dereplicated FASTA — swarm strictly requires dereplicated
  input with abundance values in the sequence headers (e.g., _1000 suffixes)
- The goal is to compare clustering quality under different d thresholds rather than
  to benchmark runtime and memory between tool versions
- Datasets are small — speedup and memory ratios measured on tera-scale inputs (10.6M
  amplicons) do not necessarily transfer to small files
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: swarm v3
  role: 'Candidate version under benchmark: cluster dereplicated amplicons with default
    d = 1, multithreaded (--threads 1/16), writing seeds via --seeds'
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline version for runtime, memory, and identical-output comparison
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Mentioned upstream tool for pair-assembling reads before dereplication and
    clustering (not part of the benchmark itself)
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:94277a2739f3eb1b766a0152fe214215abbdede376e3aea1a50b496753db4706
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v2
  - swarm v3
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# high-throughput-runtime-benchmarking

## Summary

Benchmark a new release of a high-throughput tool (swarm v3) against its predecessor (swarm v2) on million-scale dereplicated amplicon datasets, measuring wall-clock runtime and peak resident memory under 1-core and 16-core conditions while verifying that clustering output is identical.

## When to use

Use when comparing runtime speed and memory footprint of two versions of a clustering/bioinformatics tool on very large (10.6M-sequence) FASTA inputs, and when output equivalence must be verified between versions.

## When NOT to use

- Input is a raw FASTQ or non-dereplicated FASTA — swarm strictly requires dereplicated input with abundance values in the sequence headers (e.g., _1000 suffixes)
- The goal is to compare clustering quality under different d thresholds rather than to benchmark runtime and memory between tool versions
- Datasets are small — speedup and memory ratios measured on tera-scale inputs (10.6M amplicons) do not necessarily transfer to small files

## Inputs

- Dereplicated amplicon FASTA file, SSU-rRNA V4 (10.6M unique sequences, ~380 bp)
- Dereplicated amplicon FASTA file, SSU-rRNA V9 (10.6M unique sequences, ~130 bp)
- swarm v3 and swarm v2 binaries

## Outputs

- benchmark_results.csv (dataset, version, threads, runtime, peak memory, speedup, memory ratio)
- OTU seed FASTA files per run (via -w/--seeds) for v2 and v3
- Wall-clock runtime and peak resident memory measurements per (version, dataset, threads) run

## How to apply

Obtain the dereplicated amplicon FASTA datasets (V4: 10.6M unique sequences, 380 bp average; V9: 10.6M unique sequences, 130 bp average) and both tool binaries. Run each version with default parameters (swarm's local threshold d = 1) on each dataset twice, once with --threads 1 and once with --threads 16, recording wall-clock runtime and peak resident memory with an external timing utility. In every run, write cluster representatives (seeds) with the -w/--seeds option so clustering results can be compared. Compute the v3-vs-v2 speedup factor and memory-footprint ratio per dataset and thread count, and assemble all measurements into a benchmark_results.csv (dataset, version, threads, runtime, peak memory, speedup, memory ratio). Rationale: swarm v3's redesigned default algorithm targets tera-scale datasets, and the d = 1 default is justified because it yields high-resolution clustering; correctness is anchored by byte-identical seed output between versions.

## Related tools

- **swarm v3** (Candidate version under benchmark: cluster dereplicated amplicons with default d = 1, multithreaded (--threads 1/16), writing seeds via --seeds) — https://github.com/torognes/swarm
- **swarm v2** (Baseline version for runtime, memory, and identical-output comparison) — https://github.com/torognes/swarm
- **vsearch** (Mentioned upstream tool for pair-assembling reads before dereplication and clustering (not part of the benchmark itself)) — https://github.com/torognes/vsearch

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- V4 runtime speedup of swarm v3 over v2 exceeds 7x on 1 core and is approximately 10x on 16 cores
- V4 memory ratio (v3/v2 peak resident memory) falls between 0.4 and 0.6 (about half the memory of v2)
- The v3 V4 OTU seed file (-w output) is byte-identical to the v2 reference output — clustering results exactly identical
- Every (dataset, version, threads) combination produces a complete row in benchmark_results.csv with finite runtime and peak-memory values
- Sequence count preserved: all runs consume the same 10.6M unique input sequences under identical conditions

## Limitations

- A 15% slowdown was observed for swarm when using d >= 2 in certain conditions (18SV9 dataset) — benchmarks at non-default d may behave differently
- Memory footprint of swarm is roughly 0.6 times the input FASTA size, and the fastidious option can increase memory consumption significantly (controlled via -c and -y), so cap settings must be held constant across runs
- Speedup figures are specific to the tested hardware/thread conditions (1 core vs 16 cores) and dataset lengths (380 bp V4 vs 130 bp V9); they are not guaranteed to generalize

## Evidence

- [abstract] optimized CPU-usage and multithreading (more than 7 times faster with default parameters): "optimized CPU-usage and multithreading (more than 7 times faster with default parameters)"
- [abstract] reduced memory footprint: "reduced memory footprint by up to 50%"
- [methods] A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset: "A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset"
- [readme] The memory footprint of swarm is roughly 0.6 times the size of the input fasta file.: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file. When using the fastidious option, memory footprint can increase significantly."
- [readme] strict dereplication of input sequences is now mandatory: "strict dereplication of input sequences is now **mandatory**"
