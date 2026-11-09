---
name: amplicon-clustering-benchmarking
description: Use when comparing runtime and memory footprint of swarm versions on
  large dereplicated amplicon datasets, or when validating claimed speedups (e.g.,
  >5x V9, >9x V4 with fastidious mode).
when_to_use_negative:
- Input FASTA entries lack abundance annotations — swarm requires occurrence counts
  and will not work properly without dereplication
- The goal is to assess clustering quality (precision/recall of OTUs) rather than
  computational performance; this skill measures runtime and memory only
- The dataset is a small test file where runtime differences between swarm versions
  are negligible
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3293
tools:
- name: swarm v3
  role: Clustering under test; run with --fastidious and --threads 4 to benchmark
    runtime and memory against swarm v2
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline clustering version; run with identical fastidious settings and thread
    count to compute speedup and memory reduction
  repo: https://github.com/torognes/swarm
- name: GCC
  role: Compiler (g++ version 4.8.5 or more recent) used to build instrumented or
    plain swarm binaries
provenance:
  source_task_ids:
  - task_002
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:5f9d7e0900d9c1d75ef7871f5b84b06cd31dbc9e934876931a006ffebe2fda88
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - swarm v2
  - swarm v3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# amplicon-clustering-benchmarking

## Summary

Benchmark swarm clustering versions by measuring wall-clock runtime and peak resident memory on dereplicated amplicon FASTA files (e.g., SSU-rRNA V4 380 bp and V9 130 bp) with fastidious merging enabled, quantifying speedup and memory reduction of swarm v3 over swarm v2.

## When to use

Use when comparing runtime and memory footprint of swarm versions on large dereplicated amplicon datasets, or when validating claimed speedups (e.g., >5x V9, >9x V4 with fastidious mode).

## When NOT to use

- Input FASTA entries lack abundance annotations — swarm requires occurrence counts and will not work properly without dereplication
- The goal is to assess clustering quality (precision/recall of OTUs) rather than computational performance; this skill measures runtime and memory only
- The dataset is a small test file where runtime differences between swarm versions are negligible

## Inputs

- Dereplicated SSU-rRNA V4 (380 bp) amplicon FASTA file with abundance annotations in sequence headers
- Dereplicated SSU-rRNA V9 (130 bp) amplicon FASTA file with abundance annotations in sequence headers
- swarm v2 and swarm v3 executables (compiled or downloaded from torognes/swarm releases)

## Outputs

- fastidious_benchmark.csv with columns: dataset, swarm version, runtime, peak memory, speedup, memory reduction
- Per-run wall-clock runtime and peak resident memory (RSS) measurements

## How to apply

Retrieve the deposited dereplicated V4 (380 bp) and V9 (130 bp) amplicon FASTA files, each entry carrying an abundance annotation (e.g., `_1000`) as swarm requires. Obtain or compile swarm v2 and swarm v3 from the torognes/swarm repository using GNU make with GCC (g++ 4.8.5+) or clang. Run swarm v3 on each dataset with the fastidious refinement option enabled at the default local threshold d = 1 with 4 threads, then run swarm v2 with identical fastidious settings and thread count, measuring wall-clock runtime and peak resident memory (RSS) with a time/RSS profiling utility. Compute per-dataset speedup (v2 runtime / v3 runtime) and memory reduction percentage ((v2 - v3) / v2), and write results to a CSV with columns for dataset, swarm version, runtime, peak memory, speedup, and memory reduction. Validate that speedup exceeds 5x for V9 (130 bp) and 9x for V4 (380 bp), and that memory reduction falls in the 5-10% range under fastidious mode.

## Related tools

- **swarm v3** (Clustering under test; run with --fastidious and --threads 4 to benchmark runtime and memory against swarm v2) — https://github.com/torognes/swarm
- **swarm v2** (Baseline clustering version; run with identical fastidious settings and thread count to compute speedup and memory reduction) — https://github.com/torognes/swarm
- **GCC** (Compiler (g++ version 4.8.5 or more recent) used to build instrumented or plain swarm binaries)

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- Speedup (v2 runtime / v3 runtime) exceeds 5x for the V9 (130 bp) dataset
- Speedup exceeds 9x for the V4 (380 bp) dataset
- Memory reduction of v3 relative to v2 falls within the 5-10% range under fastidious mode
- fastidious_benchmark.csv exists and parses with the required columns: dataset, swarm version, runtime, peak memory, speedup, memory reduction
- Both swarm versions were run with identical settings (d = 1, fastidious, 4 threads) so the comparison is controlled

## Limitations

- Fastidious option increases memory footprint significantly compared to default mode; memory consumption can be capped with options -c and -y
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so benchmark conclusions are specific to d = 1 fastidious settings
- Swarm's memory footprint is roughly 0.6 times the size of the input fasta file in default mode, so absolute memory results depend on dataset size

## Evidence

- [results] With the fastidious merging option enabled, swarm v3 is more than 5 times faster than swarm v2: [evidence span withheld — anchor status: not_found]
- [readme] a new option called *fastidious*: "a new option called *fastidious* that refines *d* = 1 results and reduces the number of small clusters."
- [readme] memory footprint of swarm: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file. When using the fastidious option, memory footprint can increase significantly."
- [readme] Compiling swarm requires make and GCC: "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent)"
- [readme] Swarm requires each fasta entry to present a number of occurrences: "Swarm **requires** each fasta entry to present a number of occurrences to work properly. That crucial information can be produced during the [dereplication](#dereplication-mandatory) step."
