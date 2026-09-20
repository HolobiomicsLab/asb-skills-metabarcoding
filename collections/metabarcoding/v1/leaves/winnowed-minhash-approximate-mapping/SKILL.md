---
name: winnowed-minhash-approximate-mapping
description: Use when mapping long PacBio/ONT reads (≥1000 bp) against a large reference
  genome database where fast approximate mapping locations and identity estimates
  suffice and full alignments are unnecessary or too slow.
when_to_use_negative:
- Samples with a high proportion of short reads — reads below the minimum length end
  up with minimizer sets too small to reliably determine mapping locations and remain
  unassigned
- When exact base-to-base alignments are required — MashMap/MetaMaps estimate identity
  from k-mer Jaccard similarity and do not compute explicit alignments
- Complex samples comprising many non-characterized (novel) entities absent from the
  reference database
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: 'Implements the minimizer-based approximate mapping stage: builds the reference
    minimizer index, computes read minimizer sets, and emits candidate mappings via
    mapDirectly'
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying winnowed-minhash algorithm on which MetaMaps' approximate mapping
    is based; estimates sequence identity from MinHash Jaccard similarity
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:53f757be495a4f160af6dd32a279cbfb3b33a7c477ec2999fffe4f78b1821854
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
yes_rate: 0.9090909090909091
n_claims: 11
---

# winnowed-minhash-approximate-mapping

## Summary

Generates candidate mapping locations for each long read by intersecting winnowed-minimizer sets between reads and a reference database (MashMap-style winnowed-minhash strategy), estimating alignment identity from MinHash/Jaccard similarity instead of computing full alignments.

## When to use

Use when mapping long PacBio/ONT reads (≥1000 bp) against a large reference genome database where fast approximate mapping locations and identity estimates suffice and full alignments are unnecessary or too slow.

## When NOT to use

- Samples with a high proportion of short reads — reads below the minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- When exact base-to-base alignments are required — MashMap/MetaMaps estimate identity from k-mer Jaccard similarity and do not compute explicit alignments
- Complex samples comprising many non-characterized (novel) entities absent from the reference database

## Inputs

- Long-read FASTQ file (PacBio/Oxford Nanopore), reads ≥1000 bp
- Reference genome FASTA database (e.g. comprehensive RefSeq with >12,000 genomes, or miniSeq+H)
- Minimum read length parameter (1000 bp) and minimum alignment identity threshold (80%)

## Outputs

- approximate_mappings.tsv: per-read candidate mapping locations with estimated alignment identities (tab-separated)
- Candidate mappings consumed downstream by metamaps classify (e.g. classification_results.EM)

## How to apply

Build a minimizer index over the comprehensive RefSeq reference database (>12,000 genomes) using MetaMaps' index-building utility, with minimizer density auto-tuned to the configured minimum read length of 1000 bp. Compute the minimizer set for each input FASTQ read, retaining only reads that meet the minimum length of 1000 bp; shorter reads yield minimizer sets too small to reliably determine mapping locations. Intersect read minimizer sets with reference minimizer sets using the winnowed-minhash strategy (MashMap-based) to produce a list of candidate mapping locations per read, then estimate alignment identity at each candidate via the winnowed-minhash model, retaining locations with estimated identity of at least 80%. Write the per-read candidates to approximate_mappings.tsv; these mappings then feed MetaMaps' probabilistic scoring and EM-based composition estimation. Rationale: approximate mapping based on minimizer k-mer selection is much faster than alignment-based methods and can be tuned to read length and expected identity, and it avoids false positives from large contaminant genomes that plague k-mer-spectra classifiers.

## Related tools

- **MetaMaps** (Implements the minimizer-based approximate mapping stage: builds the reference minimizer index, computes read minimizer sets, and emits candidate mappings via mapDirectly) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying winnowed-minhash algorithm on which MetaMaps' approximate mapping is based; estimates sequence identity from MinHash Jaccard similarity) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results
```

## Evaluation signals

- approximate_mappings.tsv exists, is tab-separated, and each entry carries read ID, reference location, and estimated identity fields
- Every retained read entry reports candidate locations with estimated alignment identity ≥80%
- Reads shorter than the 1000 bp minimum length are absent from the output (they end up unassigned by design)
- Mappings for one read show a plausible set of candidate locations consistent with the minimizer index (subsequently resolvable by the EM step, whose mapping qualities sum to 1)

## Limitations

- Reads falling below the chosen minimum length produce minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed
- Complex samples with many non-characterized entities remain challenging; database-sample mismatches (uncharacterized taxonomic entities) are an unresolved concern
- Memory-efficient mapping via --maxmemory is heuristic and actual usage will fluctuate and can exceed the target

## Evidence

- [intro] First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29.: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [discussion] Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations.: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [readme] The approximate mapping algorithm employed by MetaMaps is based on MashMap.: "The approximate mapping algorithm employed by MetaMaps is based on [MashMap](https://github.com/marbl/MashMap)."
- [readme] estimates an unbiased k-mer based Jaccard similarity using a combination of minmers and MinHash. This is then converted to an estimate of sequence identity: [evidence span withheld — anchor status: not_found]
- [intro] reliance on approximate mapping makes the algorithm much faster than alignment-based methods, and our mapping algorithm can be tuned to different read lengths and qualities.: "reliance on approximate mapping makes the algorithm much faster than alignment-based methods, and our mapping algorithm can be tuned to different read lengths and qualities."
