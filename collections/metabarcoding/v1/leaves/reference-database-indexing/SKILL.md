---
name: reference-database-indexing
description: Use when long PacBio/Oxford Nanopore metagenomic reads must be mapped
  to a large reference database (e.g., miniSeq+H with >12,000 genomes) and speed/memory
  constraints call for minimizer-based approximate mapping rather than exact alignment.
when_to_use_negative:
- 'The sample contains a high proportion of short reads: reads below the minimum length
  end up with minimizer sets too small to reliably determine mapping locations and
  remain unassigned'
- 'Exact base-to-base alignments are required: MashMap/MetaMaps approximate mapping
  does not compute alignments explicitly, only alignment boundaries and identity estimates'
- 'Real-time/streaming analysis is needed (e.g., clinical applications during sequencing):
  MetaMaps does not support streaming data'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaMaps
  role: Builds the reference minimizer index (auto-tuned to the 1000 bp minimum read
    length) and performs the first-stage approximate mapping of long reads, writing
    candidate locations with estimated identities
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: 'Underlying approximate mapping algorithm: winnowed-minhash/minimizer k-mer
    selection with automatically determined sampling rate from minimum alignment length
    and identity thresholds'
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:7b2aa1b5df3ba24592ff888a51e08a13b9287ef664e95537176c617ea32413be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# reference-database-indexing

## Summary

Builds a minimizer-based index over a comprehensive reference genome database (e.g., >12,000 RefSeq genomes) so long PacBio/ONT reads can be approximately mapped to candidate locations before probabilistic scoring, using MetaMaps' MashMap-derived winnowed-minhash strategy.

## When to use

Use when long PacBio/Oxford Nanopore metagenomic reads must be mapped to a large reference database (e.g., miniSeq+H with >12,000 genomes) and speed/memory constraints call for minimizer-based approximate mapping rather than exact alignment.

## When NOT to use

- The sample contains a high proportion of short reads: reads below the minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Exact base-to-base alignments are required: MashMap/MetaMaps approximate mapping does not compute alignments explicitly, only alignment boundaries and identity estimates
- Real-time/streaming analysis is needed (e.g., clinical applications during sequencing): MetaMaps does not support streaming data

## Inputs

- Reference genome database FASTA (e.g., miniSeq+H: >12,000 microbial genomes plus human reference)
- Taxonomy directory accompanying the database FASTA
- Long-read FASTQ file (PacBio/Oxford Nanopore), reads >= 1000 bp retained
- Configuration parameters: minimum read length (1000 bp) and minimum alignment identity (80%)

## Outputs

- approximate_mappings.tsv: per-read candidate mapping locations (read ID, reference/contig location, estimated identity), tab-separated
- In-memory minimizer index over the reference database consumed by metamaps mapDirectly/classify

## How to apply

First construct the reference database (e.g., download RefSeq/Genbank branches with MetaMaps' downloadRefSeq.pl, such as --targetBranches archaea,bacteria,fungi) and place it under databases/ (e.g., miniSeq+H). Run MetaMaps' index-building/mapDirectly step so a minimizer index is built over the reference FASTA, with minimizer density auto-tuned to the configured minimum read length of 1000 bp and the minimum alignment identity (80%). Compute the minimizer set for each input read (discarding reads below the minimum length), then intersect read minimizer sets with reference minimizer sets via the winnowed-minhash (MashMap-based) strategy to produce candidate mapping locations per read. The k-mer sampling rate is automatically determined from the given minimum local alignment length and identity thresholds, which also controls runtime and memory; use --maxmemory (e.g., 20G on a 32G machine, ~70% of RAM) for memory-efficient mapping. Retain candidate locations whose winnowed-minhash-estimated alignment identity is at least 80% and write them to approximate_mappings.tsv, which feeds the subsequent probabilistic scoring and EM classification stages.

## Related tools

- **MetaMaps** (Builds the reference minimizer index (auto-tuned to the 1000 bp minimum read length) and performs the first-stage approximate mapping of long reads, writing candidate locations with estimated identities) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate mapping algorithm: winnowed-minhash/minimizer k-mer selection with automatically determined sampling rate from minimum alignment length and identity thresholds) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 20
```

## Evaluation signals

- approximate_mappings.tsv exists, is tab-separated, and each entry carries read ID, reference location, and estimated identity fields
- Every retained read entry reports candidate locations with estimated alignment identity of at least 80%
- Reads shorter than the configured minimum length (1000 bp) end up unassigned rather than spuriously mapped
- Memory consumption stays near the --maxmemory target (recommend ~70% of available RAM, e.g., 20G on a 32G machine); the full >12,000-genome index maps in <16 GB of RAM
- Mapping qualities of all mappings for one read sum up to 1 in the downstream classification_results.EM output

## Limitations

- Reads below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- No support for streaming data, which would be important for clinical applications and real-time sequencing control
- The --maxmemory feature is implemented heuristically; actual memory usage will fluctuate and exceed the target

## Evidence

- [intro] First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29.: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [readme] The approximate mapping algorithm employed by MetaMaps is based on MashMap.: "The approximate mapping algorithm employed by MetaMaps is based on [MashMap](https://github.com/marbl/MashMap). MetaMaps adds a mapping quality model and EM-based estimation of sample composition."
- [readme] An appropriate k-mer sampling rate is automatically determined using the given minimum local alignment length and identity thresholds.: [evidence span withheld — anchor status: not_found]
- [readme] You can use the '--maxmemory' parameter to specify a target for maximum memory consumption (in gigabytes).: [evidence span withheld — anchor status: not_found]
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
