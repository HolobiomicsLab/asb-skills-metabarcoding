---
name: binomial-probabilistic-alignment-scoring
description: Use when long reads (PacBio/ONT FASTQ) already have candidate approximate
  mappings from a minimizer-based stage and you need per-location posterior mapping
  qualities plus binomial-approximation alignment identities before EM-based composition
  estimation.
when_to_use_negative:
- 'Reads below the chosen minimum length: their minimizer sets are too small to reliably
  determine mapping locations, so the binomial scoring model has no reliable input'
- Samples dominated by short reads — MetaMaps' minimizer-based approximate mapping
  and this scoring stage are specifically designed for long reads
- Exact base-level alignment identity is required — the binomial stage yields an approximate
  k-mer-based identity, not an explicit alignment
edam_operation: http://edamontology.org/operation_2520
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaMaps
  role: Implements the binomial mapping-quality model over candidate mappings and
    emits normalized per-read posteriors plus EM-based composition estimates
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Minimizer/MinHash-based approximate mapping that supplies the candidate mapping
    locations and MinHash intersection/union sizes scored in the second stage
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:ffccb01bad782f6b53ef2044379cad5f6b6060854de060f3148507adf8649f87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_2520
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# binomial-probabilistic-alignment-scoring

## Summary

Scores each candidate mapping location of a long read with MetaMaps' second-stage probabilistic model: the MinHash intersection size is modeled as Binomial(n = union size, p = (1-e)^k), where e is derived from the top hit's estimated alignment identity, yielding posterior mapping qualities that sum to 1 per read.

## When to use

Use when long reads (PacBio/ONT FASTQ) already have candidate approximate mappings from a minimizer-based stage and you need per-location posterior mapping qualities plus binomial-approximation alignment identities before EM-based composition estimation.

## When NOT to use

- Reads below the chosen minimum length: their minimizer sets are too small to reliably determine mapping locations, so the binomial scoring model has no reliable input
- Samples dominated by short reads — MetaMaps' minimizer-based approximate mapping and this scoring stage are specifically designed for long reads
- Exact base-level alignment identity is required — the binomial stage yields an approximate k-mer-based identity, not an explicit alignment

## Inputs

- Long-read FASTQ file (PacBio/ONT reads)
- Candidate mapping locations from the minimizer-based approximate mapping stage (MetaMaps mapDirectly / MashMap output)
- MinHash intersection and union sizes per read–reference-window pair
- MetaMaps classification database (e.g. miniSeq+H with DB.fa and taxonomy)

## Outputs

- classification_results.EM: per-mapping read ID, coordinates, strand, contig, Poisson-model identity, MinHash intersection/union sizes, binomial-approximation identity, and mapping quality (qualities per read sum to 1)
- mapping_qualities.tsv: one row per read and candidate location with posterior mapping quality and estimated alignment identity
- Per-read posterior scores P_r(i) normalized to sum to 1 across candidate locations, flagging ambiguous mappings

## How to apply

After the MashMap-based mapDirectly stage produces candidate mapping locations per read together with the minimizer sets S_{r∪i} for each read–reference-window pair, count the observed shared minimizers between read and reference window. Model the shared-minimizer count as Binomial(n = |S_{r∪i}|, p = (1-e)^k), setting the per-base error rate e to 1 minus the estimated alignment identity of the read's top-scoring hit (the Poisson-model identity in the EM output). Convert per-location binomial probabilities into unnormalized posterior scores and normalize across all candidate locations so the mapping qualities for each read sum to 1. Emit per-read, per-location posteriors and estimated identities; ambiguous reads are marked by probability split across alternative locations. These scores then feed MetaMaps' EM algorithm for sample composition estimation and disambiguation of closely related genomes.

## Related tools

- **MetaMaps** (Implements the binomial mapping-quality model over candidate mappings and emits normalized per-read posteriors plus EM-based composition estimates) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Minimizer/MinHash-based approximate mapping that supplies the candidate mapping locations and MinHash intersection/union sizes scored in the second stage) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify -t 5 --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- For every read, the normalized posteriors P_r(i) across candidate locations sum to 1 (README: 'The mapping qualities of all mappings for one read sum up to 1')
- classification_results.EM exists and contains, for each mapping, the MinHash intersection size, MinHash union size, estimated alignment identity using a binomial approximation, and mapping quality fields
- Estimated per-base error e equals 1 minus the top hit's estimated alignment identity, and binomial identities are consistent with the reported Poisson-model identities
- Ambiguous reads show probability mass split across multiple alternative locations rather than a single dominant posterior
- All classified reads appear in the per-read output with both mapping-quality and identity fields populated

## Limitations

- Reads below the minimum length threshold end up with minimizer sets too small for reliable mapping and remain unassigned
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- Complex samples with many non-characterized entities and database-sample mismatches remain challenging; the binomial model cannot flag absent reference genomes by itself

## Evidence

- [intro] two-stage scoring model: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [readme] output fields: [evidence span withheld — anchor status: not_found]
- [readme] normalization invariant: [evidence span withheld — anchor status: not_found]
- [readme] role of quality model: "MetaMaps adds a mapping quality model and EM-based estimation of sample composition."
