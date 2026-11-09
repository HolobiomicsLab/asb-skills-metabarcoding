---
name: minimizer-index-construction
description: Use when PacBio/ONT long reads (minimum length 1000 bp) must be mapped
  to a comprehensive reference database and approximate candidate mapping locations
  with estimated alignment identities are needed before probabilistic scoring.
when_to_use_negative:
- 'Samples with a high proportion of short reads: reads below the minimum length (1000
  bp) yield minimizer sets too small to reliably determine mapping locations and end
  up unassigned'
- When exact base-to-base alignments are required — MashMap/MetaMaps estimate identity
  from k-mer Jaccard similarity and do not compute alignments explicitly
- When candidate mapping locations are already available (e.g. from a prior mapping
  run feeding metamaps classify)
edam_operation: http://edamontology.org/operation_0324
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Provides the index-building utility and mapDirectly step that construct the
    minimizer index and generate per-read approximate mapping candidates
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying approximate-mapping algorithm whose winnowed-minhash/minmer k-mer
    sampling MetaMaps adopts for candidate-location generation and identity estimation
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:cdb7031a0485a1d8ef50b015f9f02fda5a14caa807e3bd11b68e93fcb8f13a2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0324
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# minimizer-index-construction

## Summary

Generates candidate mapping locations for long reads against a large reference database (e.g. >12,000 RefSeq genomes) by indexing reference minimizers and intersecting them with read minimizers via a winnowed-minhash (MashMap-based) strategy, the first stage of MetaMaps' two-stage procedure.

## When to use

Use when PacBio/ONT long reads (minimum length 1000 bp) must be mapped to a comprehensive reference database and approximate candidate mapping locations with estimated alignment identities are needed before probabilistic scoring.

## When NOT to use

- Samples with a high proportion of short reads: reads below the minimum length (1000 bp) yield minimizer sets too small to reliably determine mapping locations and end up unassigned
- When exact base-to-base alignments are required — MashMap/MetaMaps estimate identity from k-mer Jaccard similarity and do not compute alignments explicitly
- When candidate mapping locations are already available (e.g. from a prior mapping run feeding metamaps classify)

## Inputs

- Long-read FASTQ file (PacBio or Oxford Nanopore, reads >= 1000 bp)
- Reference database FASTA (e.g. RefSeq-based miniSeq+H database with >12,000 genomes)
- MetaMaps index-building utility configuration (minimum read length, identity threshold)

## Outputs

- Minimizer index over the reference database
- approximate_mappings.tsv: per-read candidate mapping locations with estimated alignment identities (tab-separated)

## How to apply

Build a minimizer index over the reference FASTA (e.g. the comprehensive RefSeq database of >12,000 genomes) using MetaMaps' index-building utility, with the minimizer (k-mer) density auto-tuned to the configured minimum read length of 1000 bp. Compute the minimizer set for each input long read, retaining only reads meeting the minimum length. Intersect read-selected minimizer sets with reference-selected minimizer sets using the winnowed-minhash strategy inherited from MashMap to produce a per-read list of candidate mapping locations. Estimate alignment identity at each candidate location with the winnowed-minhash model and retain locations with estimated identity of at least 80%. Write per-read candidate locations and identities to approximate_mappings.tsv, which feeds MetaMaps' downstream probabilistic scoring and EM-based composition estimation.

## Related tools

- **MetaMaps** (Provides the index-building utility and mapDirectly step that construct the minimizer index and generate per-read approximate mapping candidates) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate-mapping algorithm whose winnowed-minhash/minmer k-mer sampling MetaMaps adopts for candidate-location generation and identity estimation) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results
```

## Evaluation signals

- approximate_mappings.tsv exists and is tab-separated with read ID, reference location, and estimated identity fields
- Every retained read entry reports candidate locations with estimated identity >= 80%
- Reads below the 1000 bp minimum length are absent from the candidate mapping output
- Index density/parameters reflect the configured minimum read length (1000 bp) and identity threshold rather than defaults for other read regimes
- Full MetaMaps run on a comprehensive RefSeq database completes in < 16 GB of RAM on a laptop-class machine

## Limitations

- Reads that fall below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Mapping cannot be tuned arbitrarily: minimizer density is auto-tuned to assumptions about read length and minimum alignment identity, so misconfigured parameters degrade candidate recall
- No support for streaming data, limiting clinical/real-time sequencing use; memory targets via --maxmemory are heuristic and actual usage fluctuates

## Evidence

- [intro] First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29.: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [intro] reliance on approximate mapping makes the algorithm much faster than alignment-based methods, and our mapping algorithm can be tuned to different read lengths and qualities.: "reliance on approximate mapping makes the algorithm much faster than alignment-based methods, and our mapping algorithm can be tuned to different read lengths and qualities."
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [readme] The approximate mapping algorithm employed by MetaMaps is based on MashMap.: "The approximate mapping algorithm employed by MetaMaps is based on [MashMap](https://github.com/marbl/MashMap)."
- [readme] Mashmap computes alignment boundaries and identity estimates using k-mers... An appropriate k-mer sampling rate is automatically determined using the given minimum local alignment length and identity thresholds.: [evidence span withheld — anchor status: not_found]
