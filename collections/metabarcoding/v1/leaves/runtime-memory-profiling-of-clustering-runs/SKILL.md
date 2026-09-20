---
name: runtime-memory-profiling-of-clustering-runs
description: Use when comparing amplicon clustering tool versions for scalability
  — e.g. deciding whether swarm v2's time/memory requirements limit very large datasets,
  and measuring how swarm v3 improves runtime with fastidious merging.
when_to_use_negative:
- Input FASTA entries lack abundance annotations — strict dereplication with occurrence
  counts is mandatory for swarm, so profiling runs will not work properly without
  them
- Clustering is run with default parameters only — the >7x default-parameter speedup
  is a separate benchmark from the fastidious-mode comparison this skill targets
- The goal is comparing cluster output quality (OTU composition) rather than runtime
  and memory scalability
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0196
tools:
- name: swarm v3
  role: Amplicon clustering run being benchmarked; run with --fastidious --threads
    4 on V4 and V9 datasets while profiling runtime and RSS
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline amplicon clustering version run with identical fastidious settings
    to compute speedup and memory-reduction ratios
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:44a309eb511cd122d571df467e610d63027f747898401e89d879751751445ccc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v2
  - swarm v3
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# Runtime-memory-profiling-of-clustering-runs

## Summary

Benchmark wall-clock runtime and peak resident memory of swarm v2 vs swarm v3 clustering runs with the fastidious option enabled, to quantify the speedup (validated as >5x for SSU-rRNA V9 130 bp and >9x for V4 380 bp) and memory reduction (5-10%) on dereplicated amplicon FASTA inputs.

## When to use

Use when comparing amplicon clustering tool versions for scalability — e.g. deciding whether swarm v2's time/memory requirements limit very large datasets, and measuring how swarm v3 improves runtime with fastidious merging.

## When NOT to use

- Input FASTA entries lack abundance annotations — strict dereplication with occurrence counts is mandatory for swarm, so profiling runs will not work properly without them
- Clustering is run with default parameters only — the >7x default-parameter speedup is a separate benchmark from the fastidious-mode comparison this skill targets
- The goal is comparing cluster output quality (OTU composition) rather than runtime and memory scalability

## Inputs

- Dereplicated SSU-rRNA V4 (380 bp) amplicon FASTA file with abundance annotations per entry
- Dereplicated SSU-rRNA V9 (130 bp) amplicon FASTA file with abundance annotations per entry
- swarm v2 and swarm v3 executables (from torognes/swarm, compiled with g++ >= 4.8.5 or clang >= 9)

## Outputs

- fastidious_benchmark.csv with columns: dataset, swarm version, runtime, peak memory, speedup, memory reduction
- Per-run wall-clock runtime and peak resident memory (RSS) measurements for each swarm version and dataset

## How to apply

Retrieve dereplicated V4 (380 bp) and V9 (130 bp) amplicon FASTA files where each entry carries a swarm-required abundance annotation (e.g. '>seqID_1000'). Run swarm v3 and swarm v2 from the torognes/swarm repository on each dataset with identical fastidious settings (local threshold d = 1, --threads 4), profiling wall-clock runtime and peak resident memory (RSS) with a time/RSS utility. Compute per-dataset speedup as v2 runtime divided by v3 runtime, and memory reduction as (v2 footprint minus v3 footprint) relative to v2. Write measurements to fastidious_benchmark.csv with columns for dataset, swarm version, runtime, peak memory, speedup, and memory reduction. The rationale is that fastidious merging is the recommended refinement for d = 1 because it reduces small clusters, but it can inflate memory use, so runtime and RSS must be profiled together.

## Related tools

- **swarm v3** (Amplicon clustering run being benchmarked; run with --fastidious --threads 4 on V4 and V9 datasets while profiling runtime and RSS) — https://github.com/torognes/swarm
- **swarm v2** (Baseline amplicon clustering version run with identical fastidious settings to compute speedup and memory-reduction ratios) — https://github.com/torognes/swarm

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- Measured speedup exceeds 5x for V9 (130 bp) and exceeds 9x for V4 (380 bp) when comparing v2 runtime to v3 runtime with fastidious enabled
- Measured memory reduction (v2 minus v3 footprint relative to v2) falls within the 5-10% range
- Both versions were run with identical settings: local threshold d = 1, fastidious option enabled, and the same thread count (4 threads)
- fastidious_benchmark.csv contains rows for every dataset x version combination with non-null runtime and peak-memory columns
- Input FASTA files are dereplicated and every entry ends with an occurrence-count annotation (e.g. _1000), as swarm requires

## Limitations

- Memory footprint can increase significantly when using the fastidious option; options -c and -y exist to control and cap swarm's memory consumption
- The fastidious option is recommended specifically for d = 1 to reduce the number of small clusters; using large d values (e.g. converting a 97% global threshold) is not a correct use of swarm
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so profiled runtimes are parameter-dependent
- Benchmark validity depends on matching thread count and fastidious settings exactly across the two versions being compared

## Evidence

- [results] fastidious benchmark speedup: [evidence span withheld — anchor status: not_found]
- [methods] swarm command line: "./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta"
- [readme] fastidious memory caveat: "When using the fastidious option, memory footprint can increase significantly. See options `-c` and `-y` to control and cap swarm's memory consumption."
- [readme] abundance annotation requirement: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [abstract] memory reduction finding: "reduced memory footprint by up to 50%"
