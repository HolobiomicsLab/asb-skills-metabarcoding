---
name: iterative-likelihood-convergence-monitoring
description: Use when probabilistic per-read mapping scores span multiple candidate
  (and possibly closely related) reference genomes and you must simultaneously estimate
  overall sample composition and disambiguate ambiguous read-to-genome assignments.
when_to_use_negative:
- The sample has a high proportion of short reads — minimizer-based mapping yields
  unreliable mapping locations below the minimum read length, so EM posteriors are
  poorly grounded
- The sample comprises many non-characterized entities absent from the database —
  EM convergence will not detect missing taxa, and database-sample mismatches require
  separate visual or automatic thresholding of coverage/identity patterns
- You only need a fast k-mer taxonomic label without composition estimation or per-read
  positional output — k-mer tools (Kraken2, CLARK, MetaOthello) may suffice, though
  they can suffer false positives from large contaminant genomes
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: MetaMaps
  role: Implements the two-stage pipeline whose classify step runs the EM algorithm
    for sample composition estimation and maximum-likelihood read disambiguation
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Provides the minimizer-based approximate mapping algorithm that generates
    the candidate mapping locations fed into the EM scoring stage
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:cf8ae39b7c923549369f27a25e72ed04a00c5a3ea083a7c4937423336611e0df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Iterative likelihood convergence monitoring for EM-based sample composition estimation

## Summary

Applies the Expectation-Maximization algorithm to iteratively re-estimate the sample composition vector F from composition-weighted read posteriors, using convergence of the total data likelihood as the stopping criterion, and resolves each ambiguous read to its maximum-likelihood genome under the converged composition. In MetaMaps this is what turns mapping-quality-based probabilities over >12,000 RefSeq genomes into both composition estimates and per-read taxonomic assignments.

## When to use

Use when probabilistic per-read mapping scores span multiple candidate (and possibly closely related) reference genomes and you must simultaneously estimate overall sample composition and disambiguate ambiguous read-to-genome assignments.

## When NOT to use

- The sample has a high proportion of short reads — minimizer-based mapping yields unreliable mapping locations below the minimum read length, so EM posteriors are poorly grounded
- The sample comprises many non-characterized entities absent from the database — EM convergence will not detect missing taxa, and database-sample mismatches require separate visual or automatic thresholding of coverage/identity patterns
- You only need a fast k-mer taxonomic label without composition estimation or per-read positional output — k-mer tools (Kraken2, CLARK, MetaOthello) may suffice, though they can suffer false positives from large contaminant genomes

## Inputs

- Probabilistic mapping scores and mapping qualities for all classified long reads (from the MetaMaps mapDirectly/mapping stage)
- Candidate mapping locations per read against a reference database (e.g. miniSeq+H: >12,000 RefSeq microbial genomes plus the human reference)
- Classification database with extended taxonomy (DB directory containing DB.fa and taxonomy subdirectory)

## Outputs

- Converged sample composition vector with estimated taxon frequencies (classification_results.EM.WIMP with Absolute, EMFrequency, PotFrequency columns)
- Per-read maximum-likelihood genome assignments (classification_results.EM.reads2Taxon and .krona variant)
- Per-read mappings with normalized mapping qualities (mapping qualities of all mappings for one read sum to 1)

## How to apply

Initialize the composition vector F uniformly across the reference database genomes (e.g. the >12,000-genome miniSeq+H RefSeq database). In the E-step, reweight each read's mapping-quality-based probabilities P_r(i) over its candidate mapping locations into composition-dependent posteriors P_r(i;F) = P_r(i)·(1/E_{r;g})·F_g using the current F. In the M-step, re-estimate F from the summed posteriors and recompute the total data likelihood, then repeat E- and M-steps until the likelihood converges — convergence is the explicit stopping criterion. Under the converged F, assign each read to its maximum-likelihood genome; in MetaMaps this EM level is the strain-level 'definedGenomes' level, and results are emitted as the .EM.WIMP composition file and .EM.reads2Taxon per-read assignment file. Accept the estimate only if it meets the reported benchmark accuracy of r2 > 0.97 for sample composition estimation.

## Related tools

- **MetaMaps** (Implements the two-stage pipeline whose classify step runs the EM algorithm for sample composition estimation and maximum-likelihood read disambiguation) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Provides the minimizer-based approximate mapping algorithm that generates the candidate mapping locations fed into the EM scoring stage) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify -t 5 --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Total data likelihood increases monotonically across EM iterations and plateaus — likelihood convergence is the stopping criterion
- Composition estimate matches expected abundances at r2 > 0.97, the reported benchmark for sample composition estimation
- Mapping qualities of all mappings for one read sum up to 1 (normalized posterior invariant)
- Sum of reads assigned in classification_results.EM.WIMP (Absolute column) equals the number of lines in classification_results.EM.reads2Taxon
- EMFrequency values lie in [0,1] and PotFrequency equals EMFrequency after correcting for unmapped reads

## Limitations

- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species; how to best implement detection of novelty from long-read data remains an open question
- Database-sample incongruity (presence of yet-uncharacterized taxonomic entities) is an important unresolved concern; visual or automatic thresholding of coverage and identity patterns is needed for HMP- or CAMI-like mismatches
- Reads below the minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned; classification_results.EM.evidenceUnknownSpecies statistics are of limited use in their current state

## Evidence

- [intro] Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm.: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [intro] EM distinguishes closely related genomes via information borrowing: "by using the EM algorithm for borrowing information across reads13,15, MetaMaps can distinguish between closely related database genomes"
- [abstract] Reported accuracy benchmark: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [readme] EM carried out at strain-level definedGenomes level: [evidence span withheld — anchor status: not_found]
- [readme] Mapping quality normalization invariant: [evidence span withheld — anchor status: not_found]
