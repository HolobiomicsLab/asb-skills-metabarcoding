---
name: minimizer-based-approximate-mapping
description: Use when mapping long PacBio/ONT metagenomic reads against a large reference
  database (e.g. >12,000 RefSeq genomes) and you need fast candidate mapping locations
  to feed downstream probabilistic scoring, rather than base-level alignments.
when_to_use_negative:
- Sample contains a high proportion of short reads — reads below the minimum length
  get minimizer sets too small to reliably determine mapping locations and remain
  unassigned
- Base-level exact alignments are required, since MashMap 'does not compute the alignments
  explicitly' and MetaMaps outputs only approximate mapping locations
- Real-time/streaming analysis is needed — MetaMaps lacks support for streaming data
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Implements the minimizer-based approximate mapping stage ('mapDirectly') for
    long-read metagenomes, built on MashMap
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: 'Underlying approximate mapping algorithm: computes local alignment boundaries
    and identity estimates from minmers/MinHash without explicit alignments'
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:56198fc093e8f5315cf8c86ac135ca23889a40c3657e1b7a4e69856b8fe73727
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# minimizer-based-approximate-mapping

## Summary

Generate candidate mapping locations for each long metagenomic read using a minimizer-based approximate mapping strategy (MashMap) instead of exact alignment, so that a comprehensive RefSeq database can be mapped rapidly on commodity hardware.

## When to use

Use when mapping long PacBio/ONT metagenomic reads against a large reference database (e.g. >12,000 RefSeq genomes) and you need fast candidate mapping locations to feed downstream probabilistic scoring, rather than base-level alignments.

## When NOT to use

- Sample contains a high proportion of short reads — reads below the minimum length get minimizer sets too small to reliably determine mapping locations and remain unassigned
- Base-level exact alignments are required, since MashMap 'does not compute the alignments explicitly' and MetaMaps outputs only approximate mapping locations
- Real-time/streaming analysis is needed — MetaMaps lacks support for streaming data

## Inputs

- Long-read metagenomic FASTQ file (PacBio/Oxford Nanopore, e.g. input.fastq)
- Reference database FASTA (e.g. databases/miniSeq+H/DB.fa with >12,000 microbial genomes plus human reference)

## Outputs

- Candidate mapping locations per long read (approximate alignment locations, estimated alignment identities, MinHash intersection and union sizes) passed to the classification stage
- Raw mapping results file consumed by './metamaps classify'

## How to apply

Run MetaMaps' 'mapDirectly' command, whose approximate mapping algorithm is based on MashMap, against the reference FASTA of a database such as miniSeq+H (built via downloadRefSeq.pl) with a long-read input FASTQ. MashMap estimates a k-mer based Jaccard similarity using minmers (winnowing) and MinHash, converts it to sequence identity via Mash distance, and requires a minimum alignment length and identity threshold; the k-mer sampling rate is set automatically from these thresholds. For large databases, target memory with --maxmemory (recommend ~70% of available RAM, e.g. 20G on a 32G machine) and parallelize with -t. The output list of possible mapping locations per read, together with MinHash intersection/union sizes and identity estimates, is then consumed by the second-stage probabilistic mapping quality model (binomial over shared minimizers) and the EM composition estimation.

## Related tools

- **MetaMaps** (Implements the minimizer-based approximate mapping stage ('mapDirectly') for long-read metagenomes, built on MashMap) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate mapping algorithm: computes local alignment boundaries and identity estimates from minmers/MinHash without explicit alignments) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results --maxmemory 20 -t 5
```

## Evaluation signals

- Output mapping file exists for all classified reads and contains per-mapping fields including MinHash intersection size, MinHash union size, estimated alignment identity (Poisson and binomial), and mapping quality
- Mapping qualities of all mappings for one read sum to 1
- Reads below the chosen minimum length should show no reliable mapping (minimizer sets too small)
- Memory footprint stays near the --maxmemory target (e.g. ~20G on a 32G machine), noting the README warns actual usage fluctuates and may exceed the target
- Estimated identities are consistent with MashMap's k-mer/Mash-distance identity estimates for the given minimum alignment length and identity thresholds

## Limitations

- Reads that fall below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Approximate mapping trades exactness for speed: 'MashMap does not compute exact sequence alignments'; identities are estimates from Jaccard/Mash distance
- MashMap3's slightly increased automatic sampling rate improves mappings and ANI prediction at the cost of more RAM
- Large contaminant genomes and database-sample incongruities (uncharacterized entities) challenge downstream interpretation even when mapping succeeds

## Evidence

- [intro] First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29.: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [intro] reliance on approximate mapping makes the algorithm much faster than alignment-based methods, and our mapping algorithm can be tuned to different read lengths and qualities.: "reliance on approximate mapping makes the algorithm much faster than alignment-based methods, and our mapping algorithm can be tuned to different read lengths and qualities."
- [readme] The approximate mapping algorithm employed by MetaMaps is based on [MashMap](https://github.com/marbl/MashMap). MetaMaps adds a mapping quality model and EM-based estimation of sample composition.: "The approximate mapping algorithm employed by MetaMaps is based on [MashMap](https://github.com/marbl/MashMap). MetaMaps adds a mapping quality model and EM-based estimation of sample composition."
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [readme] It does not compute the alignments explicitly, but rather estimates an unbiased k-mer based Jaccard similarity: [evidence span withheld — anchor status: not_found]
