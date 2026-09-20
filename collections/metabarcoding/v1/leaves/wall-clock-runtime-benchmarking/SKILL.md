---
name: wall-clock-runtime-benchmarking
description: Use when benchmarking a tool's scalability on a very large dataset (tera-scale
  amplicon FASTA) — e.g. reporting how many minutes a clustering run takes on a fixed
  thread count and core count.
when_to_use_negative:
- The tool's output has not been validated yet — a fast run that produced a malformed
  swarms file yields a meaningless runtime claim
- Parameters or hardware vary between runs (e.g. changing d or thread count), so wall-clock
  times are not comparable
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: swarm v3
  role: 'Tool being benchmarked: de novo amplicon clustering with default parameters
    (d = 1, built-in cluster breaking) on the UniEuk dataset, multithreaded via --threads
    16'
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Upstream preprocessing tool (pair-assembly, dereplication, --relabel_sha1,
    --fasta_width 0) that produces the abundance-annotated FASTA the timed clustering
    run consumes
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:671b0a25aef15c7a6551210270f5d373ca1375b435bf92fd82e3222b3cf90fbb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - swarm v3
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# wall-clock-runtime-benchmarking

## Summary

Measures and records the wall-clock time a computational tool needs to process a large input under fixed parameters and hardware (e.g. swarm v3 clustering ~324 million UniEuk SSU-rRNA V4 sequences in 50 minutes on 16 cores), so performance claims are reproducible.

## When to use

Use when benchmarking a tool's scalability on a very large dataset (tera-scale amplicon FASTA) — e.g. reporting how many minutes a clustering run takes on a fixed thread count and core count.

## When NOT to use

- The tool's output has not been validated yet — a fast run that produced a malformed swarms file yields a meaningless runtime claim
- Parameters or hardware vary between runs (e.g. changing d or thread count), so wall-clock times are not comparable

## Inputs

- Dereplicated amplicon FASTA with abundance annotations per sequence entry (e.g. amplicons.fasta, UniEuk SSU-rRNA V4, ~324M unique sequences)
- Fixed hardware specification (e.g. 16-core system) and thread count (e.g. --threads 16)

## Outputs

- Tool output file whose completion marks the timed run (e.g. amplicons.swarms, space-separated amplicon IDs per cluster, seed first)
- Wall-clock runtime record file (e.g. runtime_record.txt) with elapsed time for the run

## How to apply

Prepare the input in the tool's required format first: for swarm, the dereplicated FASTA must carry an abundance annotation per entry (e.g. '>seqID1_1000'), produced by the mandatory dereplication step. Run the tool with fixed, documented parameters — for swarm v3 default parameters mean local threshold d = 1 with built-in cluster breaking — and pin resources explicitly (e.g. --threads 16) so the timing is attributable. Wrap the invocation with a wall-clock timer such as /usr/bin/time or shell 'time' and write the elapsed time to a runtime record file. Validate the primary output (amplicons.swarms: space-separated amplicon identifiers per cluster, most abundant amplicon first as seed) before accepting the recorded runtime, and compare it against expected ranges for the dataset size and core count (roughly 30-90 minutes for ~324M sequences on 16 cores).

## Related tools

- **swarm v3** (Tool being benchmarked: de novo amplicon clustering with default parameters (d = 1, built-in cluster breaking) on the UniEuk dataset, multithreaded via --threads 16) — https://github.com/torognes/swarm
- **vsearch** (Upstream preprocessing tool (pair-assembly, dereplication, --relabel_sha1, --fasta_width 0) that produces the abundance-annotated FASTA the timed clustering run consumes) — https://github.com/torognes/vsearch

## Examples

```
/usr/bin/time -o runtime_record.txt swarm --threads 16 -o amplicons.swarms amplicons.fasta
```

## Evaluation signals

- amplicons.swarms exists and follows the swarm output format: space-separated amplicon identifiers per cluster, most abundant amplicon first as seed
- Recorded runtime falls within the expected range for the dataset size and hardware (roughly 30-90 minutes for ~324M sequences on a 16-core system)
- The input FASTA entries all carry abundance annotations (unique identifiers ending with occurrence counts, e.g. _1000), which swarm requires
- Thread count and parameters are explicitly recorded (--threads 16, default d = 1) so the timing is reproducible
- Memory behavior is consistent with expectations (~0.6x input FASTA size without fastidious; cap with -c/-y if fastidious increases footprint)

## Limitations

- Runtime depends on thread count and the local threshold d: a 15% slowdown was observed when using d >= 2 in certain conditions, so d = 1 timings do not generalize to higher d
- Memory footprint is roughly 0.6x the input FASTA size, and can increase significantly with the fastidious option unless capped via -c and -y
- Runtimes are only comparable when input dereplication is strict, since strict dereplication of input sequences is mandatory in swarm 3.0

## Evidence

- [abstract] optimized CPU-usage and multithreading (more than 7 times faster with default parameters): "optimized CPU-usage and multithreading (more than 7 times faster with default parameters)"
- [methods] A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset: "A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset"
- [readme] Swarm requires each fasta entry to present a number of occurrences to work properly.: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [readme] The memory footprint of swarm is roughly 0.6 times the size of the input fasta file.: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
- [methods] ./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta: "./swarm \
    --fastidious \
    --threads 4 \
    --seeds cluster_representatives.fasta \
    amplicons.fasta"
