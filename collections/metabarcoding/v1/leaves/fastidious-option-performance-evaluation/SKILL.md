---
name: fastidious-option-performance-evaluation
description: Use when clustering very large dereplicated amplicon FASTA files with
  swarm --fastidious and you need to quantify runtime and memory savings of swarm
  v3 over swarm v2, or judge whether v2's resource needs become limiting.
when_to_use_negative:
- When clustering with default (non-fastidious) parameters — the article reports speedups
  for default parameters separately (>7x), and fastidious significantly increases
  swarm's memory footprint
- When input FASTA entries lack abundance annotations (dereplicated occurrence counts),
  since swarm requires them to work properly
- When the goal is clustering quality rather than performance — use swarm-tests regression/unit
  tests instead of timing benchmarks
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3293
tools:
- name: swarm v3
  role: Version under test; clusters amplicons with the fastidious refinement option
    at d = 1 and 4 threads, with modernized C++ code and multithreading
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline version run with identical fastidious settings and thread count to
    compute speedup and memory reduction
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Mentioned in the article's pipeline for pair-assembly producing the dereplicated
    amplicon FASTA inputs
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:1731f30bbe4fe9b1709ec47cf5956fc4d37dea4955e842c51b78f297c81e1eb3
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
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# fastidious-option-performance-evaluation

## Summary

A benchmarking skill for measuring how swarm's fastidious merging option affects runtime and peak memory when clustering large dereplicated amplicon datasets, comparing swarm v3 against swarm v2 on the SSU-rRNA V4 (380 bp) and V9 (130 bp) datasets.

## When to use

Use when clustering very large dereplicated amplicon FASTA files with swarm --fastidious and you need to quantify runtime and memory savings of swarm v3 over swarm v2, or judge whether v2's resource needs become limiting.

## When NOT to use

- When clustering with default (non-fastidious) parameters — the article reports speedups for default parameters separately (>7x), and fastidious significantly increases swarm's memory footprint
- When input FASTA entries lack abundance annotations (dereplicated occurrence counts), since swarm requires them to work properly
- When the goal is clustering quality rather than performance — use swarm-tests regression/unit tests instead of timing benchmarks

## Inputs

- Dereplicated SSU-rRNA V4 (380 bp) amplicon FASTA file with per-entry abundance annotations
- Dereplicated SSU-rRNA V9 (130 bp) amplicon FASTA file with per-entry abundance annotations
- swarm v2 and swarm v3 executables (compiled or from torognes/swarm releases)

## Outputs

- fastidious_benchmark.csv with columns: dataset, swarm version, runtime, peak memory, speedup, memory reduction
- Per-dataset speedup ratios (v2/v3 runtime) and memory reduction percentages

## How to apply

Retrieve the deposited dereplicated V4 (380 bp) and V9 (130 bp) amplicon FASTA files, where each entry carries an abundance annotation as swarm requires (e.g., '_1000' suffixes, or -z format). Obtain or compile swarm v2 and swarm v3 from the torognes/swarm repository (make with g++ 4.8.5+ or clang 9+). Run both versions on each dataset with identical fastidious settings — local threshold d = 1 and 4 threads — measuring wall-clock runtime and peak resident memory (RSS). Compute per-dataset speedup (v2 runtime / v3 runtime) and memory reduction percentage ((v2 − v3)/v2), writing results to fastidious_benchmark.csv. The fastidious option is used because at d = 1 it reduces the number of small clusters; the benchmark validates that v3 is >5x faster for V9 and >9x faster for V4 under fastidious, with memory reduction in the 5–10% range.

## Related tools

- **swarm v3** (Version under test; clusters amplicons with the fastidious refinement option at d = 1 and 4 threads, with modernized C++ code and multithreading) — https://github.com/torognes/swarm
- **swarm v2** (Baseline version run with identical fastidious settings and thread count to compute speedup and memory reduction) — https://github.com/torognes/swarm
- **vsearch** (Mentioned in the article's pipeline for pair-assembly producing the dereplicated amplicon FASTA inputs) — https://github.com/torognes/vsearch

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- Speedup (v2 runtime / v3 runtime) exceeds 5x for the V9 (130 bp) dataset under fastidious
- Speedup exceeds 9x for the V4 (380 bp) dataset under fastidious
- Memory reduction percentage falls within the 5–10% range
- Both versions run with identical parameters: --fastidious, d = 1, 4 threads
- fastidious_benchmark.csv contains all required columns: dataset, swarm version, runtime, peak memory, speedup, memory reduction

## Limitations

- When using the fastidious option, swarm's memory footprint can increase significantly; options -c and -y control and cap memory consumption
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so results at d = 1 do not generalize to larger d
- Swarm's memory footprint is roughly 0.6 times the size of the input fasta file, so benchmark memory results depend on input size

## Evidence

- [results] fastidious speedup finding: [evidence span withheld — anchor status: not_found]
- [methods] fastidious recommendation: "The fastidious option is recommended when using *d* = 1, as it will reduce the number of small clusters"
- [readme] fastidious memory caveat: "When using the fastidious option, memory footprint can increase significantly. See options `-c` and `-y` to control and cap swarm's memory consumption."
- [readme] abundance requirement: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [readme] compilation requirements: "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent)"
