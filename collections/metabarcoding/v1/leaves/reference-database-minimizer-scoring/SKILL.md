---
name: reference-database-minimizer-scoring
description: Use when a MetaMaps (MashMap-based) approximate mapping step has produced
  candidate mapping locations per long read and each candidate must be scored probabilistically
  to obtain mapping qualities and disambiguate alternative locations.
when_to_use_negative:
- 'Samples with a high proportion of short reads: reads below the chosen minimum length
  end up with minimizer sets too small to reliably determine their mapping locations'
- Only a compositional profile is needed and per-read positional/quality information
  is irrelevant — k-mer-spectra methods may suffice
- Complex samples comprising many non-characterized entities, where database-sample
  incongruity dominates and the model lacks an explicit genomic-divergence term
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Implements the mapping quality model and EM-based classification stage applied
    to MashMap-derived candidate mappings
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Minimizer/minmer-based approximate mapping algorithm that generates the candidate
    mapping locations scored in this stage
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:d034c58166e19421aee69d75ba788023456fbe857f84ee82b5a29a5dc6483d6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# reference-database-minimizer-scoring

## Summary

Scores each candidate long-read mapping location from the minimizer-based approximate mapping stage using a binomial model of shared MinHash intersections, converting per-location probabilities into normalized posterior mapping qualities that sum to 1 per read (as implemented in MetaMaps' second stage).

## When to use

Use when a MetaMaps (MashMap-based) approximate mapping step has produced candidate mapping locations per long read and each candidate must be scored probabilistically to obtain mapping qualities and disambiguate alternative locations.

## When NOT to use

- Samples with a high proportion of short reads: reads below the chosen minimum length end up with minimizer sets too small to reliably determine their mapping locations
- Only a compositional profile is needed and per-read positional/quality information is irrelevant — k-mer-spectra methods may suffice
- Complex samples comprising many non-characterized entities, where database-sample incongruity dominates and the model lacks an explicit genomic-divergence term

## Inputs

- Mapping results from the minimizer-based approximate mapping stage (e.g. classification_results file from metamaps mapDirectly)
- MetaMaps reference database directory (e.g. miniSeq+H, >12,000 microbial genomes plus human reference, with taxonomy subdirectory)
- Long-read FASTQ input (PacBio/Oxford Nanopore reads)

## Outputs

- classification_results.EM: per-mapping lines with read ID, mapping coordinates, strand, contig ID, estimated alignment identity (Poisson model), MinHash intersection size, MinHash union size, estimated alignment identity (binomial approximation), and mapping quality
- classification_results.EM.reads2Taxon: one line per read with assigned taxon ID
- classification_results.EM.WIMP: sample composition (Absolute, EMFrequency, PotFrequency) at different taxonomic levels
- classification_results.EM.lengthAndIdentitiesPerMappingUnit: read length and estimated identity per read

## How to apply

Run MetaMaps' classification stage on the mapping results from the minimizer-based approximate mapper (MashMap-derived). For each candidate location i of read r, use the MinHash intersection size and union size reported for the read–reference-window pair to model the shared-minimizer count as a binomial probability, with the per-base error rate set from the estimated alignment identity of the read's top-scoring hit. Convert the per-location binomial probabilities into unnormalized posteriors and normalize across all candidate locations of the read so that the mapping qualities of all mappings for one read sum to 1. The EM algorithm then borrows information across reads to distinguish closely related genomes and estimate sample composition at the definedGenomes (strain-level) node. Outputs include the .EM mapping file with MinHash intersection/union sizes, Poisson-model and binomial-approximation identity estimates, and the mapping quality per mapping.

## Related tools

- **MetaMaps** (Implements the mapping quality model and EM-based classification stage applied to MashMap-derived candidate mappings) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Minimizer/minmer-based approximate mapping algorithm that generates the candidate mapping locations scored in this stage) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- For every read, the mapping qualities of all mappings sum to 1 (README invariant on classification_results.EM)
- classification_results.EM contains the expected fields per mapping: MinHash intersection size, MinHash union size, Poisson-model identity, binomial-approximation identity, and mapping quality
- Reads are only assigned when their minimizer sets are large enough; reads below the minimum length remain unassigned
- Taxon IDs beginning with 'x' in reads2Taxon are interpretable via the extended database taxonomy in the DB's taxonomy subdirectory
- EMFrequency values in classification_results.EM.WIMP are consistent with the maximum-likelihood read assignments (Absolute counts) at the definedGenomes level

## Limitations

- Reads that fall below the chosen minimum length end up with minimizer sets too small to reliably determine their mapping locations, so they remain unassigned
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- Reliance on MinHash/approximate mapping means no base-to-base alignments are computed; identities are estimates (Poisson/binomial models)

## Evidence

- [intro] Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm.: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [readme] MetaMaps adds a mapping quality model and EM-based estimation of sample composition.: "The approximate mapping algorithm employed by MetaMaps is based on [MashMap](https://github.com/marbl/MashMap). MetaMaps adds a mapping quality model and EM-based estimation of sample composition."
- [readme] estimated alignment identity using a binomial approximation, mapping quality. The mapping qualities of all mappings for one read sum up to 1.: [evidence span withheld — anchor status: not_found]
- [readme] estimates an unbiased k-mer based Jaccard similarity using a combination of minmers: [evidence span withheld — anchor status: not_found]
