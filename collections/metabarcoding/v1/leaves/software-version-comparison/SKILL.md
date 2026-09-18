---
name: software-version-comparison
description: Use when a tool release claims performance or code-quality improvements
  and you must verify them side-by-side, e.g. runtime, peak RSS, or SoftWipe scores
  of swarm v2 vs v3 on the same datasets.
when_to_use_negative:
- The goal is to run clustering and produce OTUs or a filtered occurrence table —
  this skill benchmarks the software, not the biology
- Only one version of the tool is available; a version comparison needs both source
  trees (e.g. v2 and v3)
- The tool's inputs are not held fixed across versions (different FASTA files, thread
  counts, or d thresholds), which invalidates a like-for-like speedup comparison
edam_operation: http://edamontology.org/operation_3083
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: swarm v3
  role: Newer version under evaluation; compiled from source and timed/memory-profiled
    against v2 with --fastidious, d = 1, 4 threads
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Baseline version providing the reference runtime, peak memory, and SoftWipe
    score against which v3 is compared
  repo: https://github.com/torognes/swarm
- name: SoftWipe
  role: 'C++ code-quality scorer run with identical configuration on both source trees
    to produce global quality scores (v2: 5.2, v3: 6.6 out of 10)'
merged_aliases:
- cross-version-software-comparison
merged_alias_records:
- alias: cross-version-software-comparison
  slug: cross-version-software-comparison
  jaccard_score: 0.75
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:571bfe0f7e3c87033e6f80fe997d5ebfec899ea662bf513ee03cd5a8deb0dbe7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - SoftWipe
  - swarm v2
  - swarm v3
  edam_operation: http://edamontology.org/operation_3083
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# software-version-comparison

## Summary

Empirically benchmark two released versions of a scientific tool (here, swarm v2 vs swarm v3) under identical settings to quantify speedups, memory reductions, and code-quality gains. Grounds claims like 'more than 7 times faster' and 'reduced memory footprint by up to 50%' in reproducible measurements.

## When to use

Use when a tool release claims performance or code-quality improvements and you must verify them side-by-side, e.g. runtime, peak RSS, or SoftWipe scores of swarm v2 vs v3 on the same datasets.

## When NOT to use

- The goal is to run clustering and produce OTUs or a filtered occurrence table — this skill benchmarks the software, not the biology
- Only one version of the tool is available; a version comparison needs both source trees (e.g. v2 and v3)
- The tool's inputs are not held fixed across versions (different FASTA files, thread counts, or d thresholds), which invalidates a like-for-like speedup comparison

## Inputs

- Clean source checkouts of swarm v2 and swarm v3 from the torognes/swarm GitHub repository
- Dereplicated SSU-rRNA V4 (380 bp) and V9 (130 bp) amplicon FASTA files, each entry ending with an abundance annotation (e.g. _1000)
- Toolchain: make, GCC (g++ >= 4.8.5) or clang (>= 9)
- SoftWipe tool (Zapletal et al., 2020) and a time/RSS profiling utility

## Outputs

- fastidious_benchmark.csv with columns for dataset, swarm version, runtime, peak memory, speedup, and memory reduction
- softwipe_scores.csv with one (version, global code quality score) record per version
- Speedup and memory-reduction values compared against reference expectations (e.g. >5x on V9, >9x on V4)

## How to apply

Obtain clean checkouts of both versions from the same repository (torognes/swarm) and compile each with a fixed toolchain — swarm requires only make with GCC (g++ >= 4.8.5) or clang (>= 9). Run both binaries on identical inputs with identical configuration: e.g. the dereplicated SSU-rRNA V4 (380 bp) and V9 (130 bp) amplicon FASTA files, each entry carrying an abundance annotation, with the fastidious option at local threshold d = 1 and 4 threads, recording wall-clock runtime and peak resident memory. Compute per-dataset speedup as v2 runtime divided by v3 runtime, and memory reduction as (v2 minus v3) relative to v2. For code quality, run SoftWipe with an identical configuration on both C++ source trees and collect one score per version. Validate against reference values: v3 more than 5x faster on V9 and more than 9x faster on V4 in fastidious mode, memory reduction in the 5-10% range, and a SoftWipe global score rising from 5.2 (v2) to 6.6 (v3) after more than 1600 warning fixes.

## Related tools

- **swarm v3** (Newer version under evaluation; compiled from source and timed/memory-profiled against v2 with --fastidious, d = 1, 4 threads) — https://github.com/torognes/swarm
- **swarm v2** (Baseline version providing the reference runtime, peak memory, and SoftWipe score against which v3 is compared) — https://github.com/torognes/swarm
- **SoftWipe** (C++ code-quality scorer run with identical configuration on both source trees to produce global quality scores (v2: 5.2, v3: 6.6 out of 10))

## Examples

```
git clone https://github.com/torognes/swarm.git; cd swarm/; make; ./swarm --fastidious --threads 4 amplicons.fasta
```

## Evaluation signals

- Fastidious-mode speedup exceeds 5x for SSU-rRNA V9 (130 bp) and exceeds 9x for SSU-rRNA V4 (380 bp)
- Memory reduction (v2 minus v3 footprint relative to v2) falls within the 5-10% range
- SoftWipe global code quality score matches 5.2/10 for swarm v2 and 6.6/10 for swarm v3 within tolerance, with v3 > v2
- Both versions were run on the same dereplicated FASTA inputs with identical fastidious settings (d = 1) and thread count (4 threads)
- Default-parameter runtime shows the documented more-than-7-times speedup from optimized CPU-usage and multithreading

## Limitations

- Fastidious option can increase swarm's memory footprint significantly; cap it with the -c and -y memory options when profiling
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so results may not generalize beyond d = 1
- SoftWipe scores reflect code quality of the C++ source trees and depend on identical tool configuration across versions

## Evidence

- [abstract] swarm v3 has modernized C++ source code: "swarm v3 has modernized C++ source code"
- [abstract] reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [abstract] optimized CPU-usage and multithreading speedup: "optimized CPU-usage and multithreading (more than 7 times faster with default parameters)"
- [readme] build requirements: "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang`"
- [readme] memory capping options: "When using the fastidious option, memory footprint can increase significantly. See options `-c` and `-y` to control and cap swarm's memory consumption."
- [methods] d >= 2 slowdown: "A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset"
