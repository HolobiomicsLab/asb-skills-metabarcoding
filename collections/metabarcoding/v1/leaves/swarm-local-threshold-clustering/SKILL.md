---
name: swarm-local-threshold-clustering
description: Use when dereplicated amplicon FASTA sequences (labels carrying occurrence
  counts) must be clustered into high-resolution OTUs without a global 97%-type threshold,
  or when benchmarking swarm v3 vs v2 on very large (10.6M+) sequence sets.
when_to_use_negative:
- Input is raw FASTQ or not yet pair-assembled, trimmed, and dereplicated — sequences
  must be dereplicated with abundance-bearing labels first (use vsearch, cutadapt
  upstream)
- You intend to translate a 97% global similarity threshold into a large d value —
  the README states this is 'not a correct use of swarm'
- Input is already a clustered OTU/feature table or occurrence table rather than sequence
  data
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3168
tools:
- name: swarm v3
  role: 'Primary clustering tool: linear-time d=1 algorithm, multithreading, ~50%
    lower memory, mandatory dereplication, --seeds output'
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: 'Reference version for equivalence testing: v3 must produce exactly identical
    clustering results'
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Upstream pair-assembly of FASTQ/SFF reads before dereplication and clustering
  repo: https://github.com/torognes/vsearch
- name: swarm-tests
  role: Black-box unit tests (806 tests) for validating input formats, option values,
    and regressions of a swarm build
  repo: https://github.com/frederic-mahe/swarm-tests
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:b8411f772294140bf395e3e144592d922fe5b403e3276b21b259d9715f07b44c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v2
  - swarm v3
  - swarm-tests
  - vsearch
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# swarm-local-threshold-clustering

## Summary

De novo amplicon clustering into OTUs using swarm's local linking threshold d (default d = 1) instead of an arbitrary global similarity threshold, with v3 providing a linear-complexity fast algorithm, ~50% memory reduction, and multithreading for tera-scale datasets.

## When to use

Use when dereplicated amplicon FASTA sequences (labels carrying occurrence counts) must be clustered into high-resolution OTUs without a global 97%-type threshold, or when benchmarking swarm v3 vs v2 on very large (10.6M+) sequence sets.

## When NOT to use

- Input is raw FASTQ or not yet pair-assembled, trimmed, and dereplicated — sequences must be dereplicated with abundance-bearing labels first (use vsearch, cutadapt upstream)
- You intend to translate a 97% global similarity threshold into a large d value — the README states this is 'not a correct use of swarm'
- Input is already a clustered OTU/feature table or occurrence table rather than sequence data

## Inputs

- Dereplicated amplicon FASTA file (e.g. 10.6M unique SSU-rRNA V4 sequences, 380 bp avg; or V9, 130 bp avg) with sequence labels ending in occurrence counts
- Choice of parameters: local threshold d (default 1), --fastidious flag, --threads, --seeds output path

## Outputs

- OTU clusters with seeds/cluster representatives in FASTA format (via -w/--seeds, sorted by decreasing abundance then label)
- Benchmark table of runtime, peak memory, speedup, and memory ratio per dataset/version/threads (benchmark_results.csv) when used for benchmarking

## How to apply

Input must be a dereplicated FASTA where each identifier ends with the sequence's occurrence count (e.g. >seqID1_1000), since strict dereplication is mandatory in swarm 3.0. Run './swarm amplicons.fasta' (default -d 1) or add '--fastidious' with d = 1 to reduce small clusters, '--threads N' to exploit v3's multithreading, and '-w/--seeds' to write cluster representatives sorted by decreasing abundance then alphabetical label. Do not convert a 97% global threshold into a large d value; d = 1 yields fine-scale, stable clusters, and d = 2 or 3 is reserved for long amplicons or shallow sequencing. Memory usage is roughly 0.6x the input FASTA size (more with --fastidious, controllable via -c and -y). Validate by confirming identical OTU outputs across versions/thread counts (v3 reproduces v2 clustering byte-identically) and, for benchmarks, expected speedups (>7x on 1 core, ~10x on 16 cores, memory ratio 0.4–0.6 on the 10.6M-sequence V4 dataset).

## Related tools

- **swarm v3** (Primary clustering tool: linear-time d=1 algorithm, multithreading, ~50% lower memory, mandatory dereplication, --seeds output) — https://github.com/torognes/swarm
- **swarm v2** (Reference version for equivalence testing: v3 must produce exactly identical clustering results) — https://github.com/torognes/swarm
- **vsearch** (Upstream pair-assembly of FASTQ/SFF reads before dereplication and clustering) — https://github.com/torognes/vsearch
- **swarm-tests** (Black-box unit tests (806 tests) for validating input formats, option values, and regressions of a swarm build) — https://github.com/frederic-mahe/swarm-tests

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- v3 and v2 OTU seed outputs are byte-identical on the same dereplicated input (e.g. V4 dataset)
- V4 benchmark speedup exceeds 7x on 1 core and is approximately 10x on 16 cores; memory ratio (v3/v2) falls between 0.4 and 0.6
- Every FASTA input label ends with an occurrence count; swarm 3.0 errors if strict dereplication has not been applied
- With --seeds, representatives are sorted by decreasing abundance, then alphabetical order of sequence labels
- Build passes the frederic-mahe/swarm-tests suite: 'bash run_all_tests.sh' (input formats, option values, fixed-bug regressions)

## Limitations

- With fastidious, memory footprint can increase significantly; use -c and -y to control/cap memory consumption
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset)
- Under rare conditions a marker may not evolve fast enough to distinguish molecular taxa, causing swarm to form large-radius clusters while classic methods cut at the 97% threshold
- swarm requires each FASTA entry to carry an occurrence count; strict dereplication is mandatory in v3

## Evidence

- [readme] swarm results are resilient to input-order changes and rely on a small local linking threshold d: "**swarm** results are
resilient to input-order changes and rely on a small **local** linking
threshold *d*"
- [readme] tests have shown that using the default d value (d = 1) gives good results on most datasets: "tests have shown that using the default *d* value (*d* = 1) gives
good results on most datasets"
- [methods] The fastidious option is recommended when using d = 1, as it will reduce the number of small clusters: "The fastidious option is recommended when using *d* = 1, as it will reduce the number of small clusters"
- [readme] The memory footprint of swarm is roughly 0.6 times the size of the input fasta file: "The memory footprint of **swarm** is roughly 0.6 times the size of the
input fasta file"
- [readme] swarm 3.0: strict dereplication of input sequences is now mandatory; --seeds output sorted by decreasing abundance then alphabetical order: "strict dereplication of input sequences is now **mandatory**"
