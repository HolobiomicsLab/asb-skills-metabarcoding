---
name: mapping-quality-posterior-normalization
description: Use when a long-read metagenomic mapper (MetaMaps after MashMap-style
  approximate mapping) has produced multiple candidate mapping locations per read
  and you need probabilistic per-location mapping qualities rather than a single highest-ranked
  hit.
when_to_use_negative:
- The sample contains a high proportion of short reads — reads below the chosen minimum
  length end up with minimizer sets too small to reliably determine mapping locations
- You need exact base-to-base alignments rather than approximate mapping locations
  with probabilistic qualities (MashMap/MetaMaps do not compute alignments explicitly)
- The goal is pure k-mer-based taxonomic classification without positional mapping
  information (e.g. Kraken2-style pipelines)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_2269
tools:
- name: MetaMaps
  role: Implements the binomial mapping-quality model over shared minimizers and normalizes
    posterior mapping qualities across candidate locations; the classify step also
    runs EM-based composition estimation
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying approximate mapping algorithm that generates candidate mapping
    locations and the minimizer-based MinHash intersection/union statistics consumed
    by the quality model
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:442875995a043f411a88d4abeda3519d2ab730d32604c5df59ac9a46aeba71cd
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
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# mapping-quality-posterior-normalization

## Summary

Scores each candidate mapping location of a long read with MetaMaps' binomial model over shared minimizers (MinHash intersection size against union size, with success probability (1-e)^k), then normalizes the per-location scores into posterior mapping qualities that sum to 1 across all candidate locations of the read.

## When to use

Use when a long-read metagenomic mapper (MetaMaps after MashMap-style approximate mapping) has produced multiple candidate mapping locations per read and you need probabilistic per-location mapping qualities rather than a single best hit.

## When NOT to use

- The sample contains a high proportion of short reads — reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations
- You need exact base-to-base alignments rather than approximate mapping locations with probabilistic qualities (MashMap/MetaMaps do not compute alignments explicitly)
- The goal is pure k-mer-based taxonomic classification without positional mapping information (e.g. Kraken2-style pipelines)

## Inputs

- Candidate mapping locations per long read from the mapDirectly approximate-mapping step (--mappings file)
- MinHash intersection size and MinHash union size for each read–reference-window pair
- Estimated alignment identity of each read's top-scoring hit (defines binomial error rate e = 1 - identity)
- MetaMaps classification database (e.g. miniSeq+H, >12,000 RefSeq microbial genomes plus human reference)

## Outputs

- Normalized posterior mapping quality per read–location pair (mapping qualities across all mappings of one read sum to 1)
- classification_results.EM: per-mapping fields including read ID, mapping coordinates, strand, contig ID, estimated alignment identity (Poisson model), MinHash intersection size, MinHash union size, estimated alignment identity (binomial approximation), and mapping quality
- Per-read taxonomic assignments and EM-based sample composition (classification_results.EM.WIMP, .EM.reads2Taxon)

## How to apply

Load the candidate mapping locations per long read produced by the minimizer-based approximate mapping stage (MashMap-based mapDirectly step), together with the MinHash intersection and union sizes for each read–reference-window pair. For each candidate location, model the observed shared-minimizer count as Binomial(n = |S_{r∪i}| (union size), p = (1-e)^k), setting the per-base error rate e to 1 minus the estimated alignment identity of the read's top-scoring hit. Convert the per-location binomial probabilities into unnormalized posterior scores and normalize across all candidate locations of the read so that the mapping qualities of all mappings for one read sum up to 1. In MetaMaps this is realized by the `classify` step, which also feeds these scores into the EM algorithm for estimating sample composition at the definedGenomes (strain) level and disambiguating closely related genomes.

## Related tools

- **MetaMaps** (Implements the binomial mapping-quality model over shared minimizers and normalizes posterior mapping qualities across candidate locations; the classify step also runs EM-based composition estimation) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate mapping algorithm that generates candidate mapping locations and the minimizer-based MinHash intersection/union statistics consumed by the quality model) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- For every read, the normalized posterior mapping qualities P_r(i) across all candidate locations sum to 1 (README: 'The mapping qualities of all mappings for one read sum up to 1')
- mapping_qualities.tsv (classification_results.EM) exists with the mapping-quality and estimated-alignment-identity fields present for all classified reads
- Per-mapping MinHash intersection size is consistent with (not greater than) the MinHash union size reported on the same line
- Binomial success probability (1-e)^k reflects the top hit's estimated alignment identity — identity estimates from the binomial approximation should be coherent with the Poisson-model identity on the same mapping
- Ambiguous mappings show probability mass split across alternative locations of the same read, while unique high-identity reads concentrate quality on a single location

## Limitations

- Reads below the minimum length requirement remain unassigned because their minimizer sets are too small to reliably determine mapping locations
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species; database-sample incongruity (uncharacterized taxonomic entities) remains an unresolved concern

## Evidence

- [intro] Each mapping location is scored probabilistically and composition estimated: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [readme] EM output fields include MinHash statistics, binomial identity, and mapping quality: [evidence span withheld — anchor status: not_found]
- [readme] Posterior mapping qualities are normalized to sum to 1 per read: [evidence span withheld — anchor status: not_found]
- [discussion] Short reads fail because minimizer sets are too small: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
