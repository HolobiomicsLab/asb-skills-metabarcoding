---
name: amplicon-otu-clustering
description: Use when you have a dereplicated amplicon FASTA (each entry labeled with
  its abundance, e.g. seqID_1000) and need high-resolution, input-order-resilient
  OTU clustering of datasets from thousands up to tera-scale (hundreds of millions
  of unique sequences).
when_to_use_negative:
- The input FASTA is not dereplicated or lacks abundance annotations in the headers
  — swarm requires strict dereplication and per-entry occurrence counts and will not
  work properly
- You intend to mimic a 97% global similarity threshold by setting a large d value
  — this is an incorrect use of swarm; clusters are naturally larger than d and small
  d values (1-3) should be used
- The raw reads have not been pair-assembled (e.g. with vsearch) and trimmed of adaptors/primers
  (e.g. with cutadapt) — preprocessing must precede dereplication and clustering
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: swarm v3
  role: Primary de novo amplicon clustering engine producing fine-scale OTUs with
    local threshold d = 1, multithreading, and reduced memory footprint
  repo: https://github.com/torognes/swarm
- name: swarm v2
  role: Reference clustering version used to validate that v3 produces identical clustering
    results (byte-identical seeds output)
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Upstream tool for pair-assembling SFF or FASTQ files before dereplication
    and clustering
  repo: https://github.com/torognes/vsearch
- name: swarm-tests
  role: Black-box unit tests (input formats, option values, fixed bugs) to validate
    clustering behaviour across swarm versions
  repo: https://github.com/frederic-mahe/swarm-tests
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:7d13b70777ed0a613dbb9b6e0ad340e42855cf7c66a9574aefe6dafab855afb8
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
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# amplicon-otu-clustering

## Summary

De novo amplicon clustering into fine-scale OTUs with swarm (v3), using the local linking threshold d (default d = 1) instead of an arbitrary global similarity threshold, on strictly dereplicated FASTA inputs with abundance annotations.

## When to use

Use when you have a dereplicated amplicon FASTA (each entry labeled with its abundance, e.g. seqID_1000) and need high-resolution, input-order-resilient OTU clustering of datasets from thousands up to tera-scale (hundreds of millions of unique sequences).

## When NOT to use

- The input FASTA is not dereplicated or lacks abundance annotations in the headers — swarm requires strict dereplication and per-entry occurrence counts and will not work properly
- You intend to mimic a 97% global similarity threshold by setting a large d value — this is an incorrect use of swarm; clusters are naturally larger than d and small d values (1-3) should be used
- The raw reads have not been pair-assembled (e.g. with vsearch) and trimmed of adaptors/primers (e.g. with cutadapt) — preprocessing must precede dereplication and clustering

## Inputs

- Dereplicated amplicon FASTA file (amplicons.fasta) with unique sequence identifiers ending in an abundance count (e.g. >seqID1_1000), or the -z alternative header format
- Optional: thread count, fastidious flag, and chosen local threshold d (command-line options)

## Outputs

- Clustering output file (*.swarms): space-separated amplicon identifiers per cluster, most abundant amplicon (seed) first
- Seeds/cluster representatives FASTA via -w/--seeds, sorted by decreasing abundance then alphabetical order
- Optional benchmark record (runtime, peak memory, speedup, memory ratio) e.g. benchmark_results.csv

## How to apply

First ensure the input amplicon FASTA is strictly dereplicated: identical sequences merged and sorted by decreasing abundance, with each FASTA header carrying an occurrence count (swarm makes dereplication mandatory). Run swarm with the local threshold d = 1 (default), optionally with --fastidious to reduce the number of small clusters and --threads N to exploit multithreading; chain-breaking is built-in and performed by default. Write cluster representatives with the -w/--seeds option (sorted by decreasing abundance, then alphabetical label order) and the full clustering with -o; for benchmarking, wrap the run in /usr/bin/time to record wall-clock runtime and peak resident memory. Keep d small (1, occasionally 2 or 3) — do not convert a 97% global threshold into a large d, as clusters produced by swarm are naturally larger than d. swarm v3 clusters 10.6M unique V4 sequences >7x faster than v2 on 1 core (~10x on 16 cores) using about half the memory, and clusters ~324M unique UniEuk V4 sequences in about 50 minutes on 16 cores.

## Related tools

- **swarm v3** (Primary de novo amplicon clustering engine producing fine-scale OTUs with local threshold d = 1, multithreading, and reduced memory footprint) — https://github.com/torognes/swarm
- **swarm v2** (Reference clustering version used to validate that v3 produces identical clustering results (byte-identical seeds output)) — https://github.com/torognes/swarm
- **vsearch** (Upstream tool for pair-assembling SFF or FASTQ files before dereplication and clustering) — https://github.com/torognes/vsearch
- **swarm-tests** (Black-box unit tests (input formats, option values, fixed bugs) to validate clustering behaviour across swarm versions) — https://github.com/frederic-mahe/swarm-tests

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- Output .swarms file follows the expected format: space-separated amplicon identifiers per cluster with the most abundant amplicon first as seed
- The -w/--seeds FASTA is sorted by decreasing abundance and then by alphabetical order of sequence labels, and for d = 1 defaults matches the swarm v2 reference output exactly (byte-identical OTUs)
- Reordering the input FASTA does not change the resulting clusters (swarm results are resilient to input-order changes)
- Performance sanity check: swarm v3 with d = 1 is more than 7x faster than v2 on 1 core (~10x on 16 cores) with roughly half the memory; ~324M UniEuk V4 sequences cluster in roughly 30-90 minutes on 16 cores
- Memory footprint is roughly 0.6x the input FASTA size (higher with --fastidious; control/cap with the -c and -y options)

## Limitations

- Using large d values (converting a 97% global threshold into a number of differences) is an incorrect use of swarm and degrades results; a 15% slowdown was observed for d >= 2 in certain conditions (18SV9 dataset)
- With d = 1 and without --fastidious, many small clusters can be produced; the fastidious option can significantly increase memory consumption
- If a marker does not evolve fast enough (rare condition), swarm may form a cluster with a large radius for abundant sequences, whereas classic methods cut at the global threshold
- Strict dereplication of input sequences is mandatory in swarm 3, adding a required preprocessing dependency

## Evidence

- [abstract] produces fine-scale molecular operational taxonomic units (OTUs) that are free of arbitrary global clustering thresholds: "produces fine-scale molecular operational taxonomic units (OTUs) that are free of arbitrary global clustering thresholds"
- [methods] The fastidious option is recommended when using *d* = 1, as it will reduce the number of small clusters: "The fastidious option is recommended when using *d* = 1, as it will reduce the number of small clusters"
- [readme] Swarm **requires** each fasta entry to present a number of occurrences to work properly.: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [readme] The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file.: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
