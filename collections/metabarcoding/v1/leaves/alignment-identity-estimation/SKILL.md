---
name: alignment-identity-estimation
description: Use when mapping long reads (minimum length ~1000 bp) to a large reference
  database (>12,000 RefSeq genomes) and you need fast, per-location approximate alignment
  identity estimates instead of exact base-to-base alignments.
when_to_use_negative:
- Sample contains a high proportion of short reads, or reads below the 1000 bp minimum
  length, since minimizer sets become too small to reliably determine mapping locations
- You need exact base-to-base alignments rather than approximate boundaries and identity
  estimates (MashMap does not compute alignments explicitly)
- Sample comprises many non-characterized entities absent or highly divergent from
  the reference database, where database-sample incongruity dominates assignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: MetaMaps
  role: Implements the minimizer-based approximate mapping and winnowed-minhash identity
    estimation over the reference database, with index-building utility and mapDirectly
    step
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: 'Provides the underlying approximate mapping algorithm: computes alignment
    boundaries and identity estimates from k-mer Jaccard similarity with automatic
    k-mer sampling-rate tuning'
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:f2a18fa3608f17ddf57b295f1150e98d2cbe74a6262df67d8ad52e83bc38f29a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# alignment-identity-estimation

## Summary

Estimate alignment identity between long metagenomic reads (PacBio/ONT) and reference database genomes using a winnowed-minhash (MinHash/Jaccard-based) approach inherited from MashMap, as used in MetaMaps' approximate mapping stage, retaining candidate locations with estimated identity of at least 80%.

## When to use

Use when mapping long reads (minimum length ~1000 bp) to a large reference database (>12,000 RefSeq genomes) and you need fast, per-location approximate alignment identity estimates instead of exact base-to-base alignments.

## When NOT to use

- Sample contains a high proportion of short reads, or reads below the 1000 bp minimum length, since minimizer sets become too small to reliably determine mapping locations
- You need exact base-to-base alignments rather than approximate boundaries and identity estimates (MashMap does not compute alignments explicitly)
- Sample comprises many non-characterized entities absent or highly divergent from the reference database, where database-sample incongruity dominates assignment

## Inputs

- Long-read FASTQ file (PacBio or Oxford Nanopore), reads >= 1000 bp
- Reference database FASTA (e.g. miniSeq+H or comprehensive RefSeq with >12,000 microbial genomes plus human reference)
- Configured minimum read length (1000 bp) and minimum alignment identity threshold (80%)

## Outputs

- Approximate mapping locations per read with estimated alignment identities (approximate_mappings.tsv / classification_results.EM), including MinHash intersection size, MinHash union size, Poisson-model identity, and binomial-approximation identity
- classification_results.EM.lengthAndIdentitiesPerMappingUnit: per-read Identity and Length fields sorted by mapped contig
- Per-read mapping quality values, with mapping qualities of all mappings for one read summing to 1

## How to apply

Build a minimizer index over the reference FASTA (e.g. the miniSeq+H or full RefSeq database) with MetaMaps' index-building utility, with minimizer density auto-tuned to the configured minimum read length (1000 bp). For each input read meeting the minimum length, compute its minimizer set and intersect it with reference-selected minimizer sets using the winnowed-minhash strategy (MashMap-based) to generate candidate mapping locations; the k-mer sampling rate is automatically determined from the minimum alignment length and identity threshold. At each candidate location, estimate alignment identity from the MinHash Jaccard estimate, reporting both a Poisson-model identity and a binomial-approximation identity along with MinHash intersection and union sizes in the mappings file. Retain candidate locations whose estimated identity meets the minimum identity of 80%, and let the downstream probabilistic scoring/EM stages use these identities for species-level assignment. Rationale: approximate mapping makes MetaMaps much faster than alignment-based methods while still yielding per-read positional and identity information, at <16 GB RAM for >12,000 genomes.

## Related tools

- **MetaMaps** (Implements the minimizer-based approximate mapping and winnowed-minhash identity estimation over the reference database, with index-building utility and mapDirectly step) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Provides the underlying approximate mapping algorithm: computes alignment boundaries and identity estimates from k-mer Jaccard similarity with automatic k-mer sampling-rate tuning) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results
```

## Evaluation signals

- Every retained read entry in approximate_mappings.tsv reports candidate locations with estimated identity >= 80%
- Output file is tab-separated with read, reference location, and estimated identity fields and exists after the mapDirectly step
- For each read, the mapping qualities of all mappings sum to 1, and each mapping line carries Poisson-model identity, binomial-approximation identity, MinHash intersection size, and MinHash union size
- Estimated identities are consistent with the reported MinHash intersection/union sizes (Jaccard-derived identity)
- Reads below the 1000 bp minimum length are absent from the mapping output (unassigned)

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed
- The mapping is approximate: MashMap computes alignment boundaries and identity estimates but does not compute explicit base-to-base alignments
- Complex samples with many non-characterized entities, and database-sample mismatches (yet-uncharacterized taxonomic entities), remain challenging

## Evidence

- [intro] Minimizer-based candidate generation and identity estimation: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [readme] MetaMaps builds on MashMap: "The approximate mapping algorithm employed by MetaMaps is based on [MashMap](https://github.com/marbl/MashMap)."
- [readme] Identity fields in the mapping output: [evidence span withheld — anchor status: not_found]
- [readme] MashMap identity estimation is k-mer based, not alignment based: [evidence span withheld — anchor status: not_found]
- [intro] Fast approximate mapping versus alignment-based methods: "reliance on approximate mapping makes the algorithm much faster than alignment-based methods, and our mapping algorithm can be tuned to different read lengths and qualities."
