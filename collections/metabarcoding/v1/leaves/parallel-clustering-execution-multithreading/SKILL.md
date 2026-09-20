---
name: parallel-clustering-execution-multithreading
description: Use when clustering very large dereplicated amplicon FASTA files (hundreds
  of millions of unique sequences) with swarm v3 on a multicore host, to parallelize
  execution via --threads and benchmark wall-clock runtime.
when_to_use_negative:
- The input FASTA is not dereplicated or sequence headers lack occurrence counts (e.g.,
  _1000) — swarm requires each fasta entry to present a number of occurrences to work
  properly.
- You need global-similarity-threshold OTU picking (e.g., 97%) — swarm relies on a
  small local linking threshold d (default d = 1), not a global clustering threshold.
- Memory-constrained host combined with --fastidious — the fastidious option can increase
  memory footprint significantly; without -c/-y caps the run may exhaust RAM.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: swarm v3
  role: Multithreaded amplicon clustering engine run with --threads 16 and default
    parameters (d = 1, built-in chain-breaking); writes amplicons.swarms via -o
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Upstream pair-assembly/dereplication step producing the abundance-annotated
    FASTA that swarm clusters
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:6bc44c0e889c2af302f2bfbc92791e7471d593b7977a5f40b0852690bb6508d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v3
  - vsearch
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# parallel-clustering-execution-multithreading

## Summary

Executes swarm v3 amplicon clustering in multithreaded mode (--threads) on dereplicated, abundance-annotated FASTA input, so that tera-scale datasets such as the UniEuk SSU-rRNA V4 set (~324 million unique sequences) cluster with default parameters (d = 1) in roughly 50 minutes on a 16-core system.

## When to use

Use when clustering very large dereplicated amplicon FASTA files (hundreds of millions of unique sequences) with swarm v3 on a multicore host, to parallelize execution via --threads and benchmark wall-clock runtime.

## When NOT to use

- The input FASTA is not dereplicated or sequence headers lack occurrence counts (e.g., _1000) — swarm requires each fasta entry to present a number of occurrences to work properly.
- You need global-similarity-threshold OTU picking (e.g., 97%) — swarm relies on a small local linking threshold d (default d = 1), not a global clustering threshold.
- Memory-constrained host combined with --fastidious — the fastidious option can increase memory footprint significantly; without -c/-y caps the run may exhaust RAM.

## Inputs

- Dereplicated amplicon FASTA (e.g., amplicons.fasta) whose headers carry occurrence/abundance annotations (e.g., >seqID1_1000)
- Multicore host (benchmark case: 16-core system)

## Outputs

- amplicons.swarms clustering file (space-separated amplicon identifiers per cluster, most abundant amplicon first as seed)
- Recorded wall-clock runtime (e.g., runtime_record; ~50 minutes for the UniEuk V4 dataset on 16 cores)

## How to apply

Load the dereplicated amplicon FASTA (amplicons.fasta) and confirm every entry carries an abundance annotation in its identifier (e.g., >seqID1_1000), since strict dereplication is mandatory in swarm v3. Execute swarm v3 with default parameters — local linking threshold d = 1 with built-in chain-breaking — activating multithreading with --threads 16 (matching available cores) and writing clusters with -o amplicons.swarms. Time the run with a wall-clock timer (e.g., /usr/bin/time or shell time) and record the elapsed time in a runtime record. The rationale is that optimized CPU-usage and multithreading make swarm v3 more than 7 times faster with default parameters, so the nearly 324M-sequence UniEuk SSU-rRNA V4 dataset clusters in about 50 minutes on 16 cores. Monitor memory: the footprint is roughly 0.6 times the input FASTA size and increases significantly with --fastidious, so use the -c and -y options to control and cap consumption.

## Related tools

- **swarm v3** (Multithreaded amplicon clustering engine run with --threads 16 and default parameters (d = 1, built-in chain-breaking); writes amplicons.swarms via -o) — https://github.com/torognes/swarm
- **vsearch** (Upstream pair-assembly/dereplication step producing the abundance-annotated FASTA that swarm clusters) — https://github.com/torognes/vsearch

## Examples

```
time ./swarm --threads 16 -o amplicons.swarms amplicons.fasta
```

## Evaluation signals

- amplicons.swarms exists and follows the swarm output format: space-separated amplicon identifiers per cluster, with the most abundant amplicon first as seed.
- Recorded wall-clock runtime for the UniEuk SSU-rRNA V4 dataset (~324M unique sequences) falls within the expected ~30–90 minute range on a 16-core system (paper: 50 minutes).
- Throughput reflects the v3 speedup: multithreaded runs with default parameters are more than 7 times faster than previous swarm versions.
- Memory consumption stays around 0.6 times the input FASTA size unless --fastidious is used, in which case -c and -y keep it bounded.

## Limitations

- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so higher d values cost runtime relative to the default d = 1.
- Memory footprint is roughly 0.6 times the input FASTA size and increases significantly with the fastidious option; use -c and -y to control and cap it.
- Speed and memory figures apply to default parameters (d = 1); long amplicons or shallow sequencing may require d = 2 or d = 3, very rarely more.

## Evidence

- [results] 16-core benchmark of tera-scale clustering: [evidence span withheld — anchor status: not_found]
- [abstract] multithreaded speedup of swarm v3: "optimized CPU-usage and multithreading (more than 7 times faster with default parameters)"
- [methods] default parameters include built-in chain-breaking: "The chain-breaking, which used to be performed in a second step in **swarm** 1.0, is now built-in and performed by default."
- [methods] higher d values slow execution: "A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset"
- [readme] memory footprint scaling: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
- [readme] abundance annotation is a hard input requirement: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
