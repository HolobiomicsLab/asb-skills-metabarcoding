---
name: composition-dependent-mapping-posterior-computation
description: Use when long reads (PacBio/ONT) have ambiguous probabilistic mapping
  scores across multiple candidate locations in a large RefSeq database and you need
  composition-aware resolution plus sample composition estimation.
when_to_use_negative:
- Samples dominated by short reads — the minimizer-based approximate mapping underlying
  the posteriors requires sufficiently long reads; short reads end up with minimizer
  sets too small for reliable mapping locations
- Samples with many non-characterized (novel) entities absent from the database —
  no explicit genomic-divergence term is in the model, so novel strains/species cannot
  be explicitly detected
- You only need raw candidate mapping locations without composition-aware disambiguation
  — the approximate mapping stage (MashMap-based) alone suffices
edam_operation: http://edamontology.org/operation_3660
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: 'Implements the EM loop: computes composition-dependent posteriors P_r(i;F),
    re-estimates F in the M-step, and emits per-read maximum-likelihood assignments
    (./metamaps classify)'
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Provides the minimizer-based approximate mapping algorithm that generates
    the candidate mapping locations the EM posteriors reweight
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:6933965234fea71eebe7de4bc4bb02a57a677cf6116e64bf2051205d2568f034
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3660
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# composition-dependent-mapping-posterior-computation

## Summary

Recomputes each long read's candidate mapping posteriors weighted by the current genome-composition vector F (P_r(i;F) = P_r(i)·(1/E_{r;g})·F_g) inside MetaMaps' EM loop, and assigns each read to its maximum-likelihood genome once F converges. This borrows information across reads to disambiguate closely related RefSeq genomes and yields the sample composition estimate.

## When to use

Use when long reads (PacBio/ONT) have ambiguous probabilistic mapping scores across multiple candidate locations in a large RefSeq database and you need composition-aware resolution plus sample composition estimation.

## When NOT to use

- Samples dominated by short reads — the minimizer-based approximate mapping underlying the posteriors requires sufficiently long reads; short reads end up with minimizer sets too small for reliable mapping locations
- Samples with many non-characterized (novel) entities absent from the database — no explicit genomic-divergence term is in the model, so novel strains/species cannot be explicitly detected
- You only need raw candidate mapping locations without composition-aware disambiguation — the approximate mapping stage (MashMap-based) alone suffices

## Inputs

- Probabilistic mapping scores and mapping qualities for all classified long reads (output of the MetaMaps mapDirectly + probabilistic scoring stage, e.g. classification_results)
- Reference database of >12,000 RefSeq microbial genomes (e.g. miniSeq+H)
- Long-read FASTQ input (PacBio/Oxford Nanopore)

## Outputs

- Converged composition vector / sample composition at definedGenome (strain) level (classification_results.EM.WIMP with Absolute, EMFrequency, PotFrequency columns)
- Per-read maximum-likelihood taxonomic assignments (classification_results.EM.reads2Taxon)
- Per-read maximum-likelihood mappings with mapping qualities that sum to 1 per read (classification_results.EM)

## How to apply

Load the mapping-quality-based probabilities P_r(i) for all classified long reads produced by MetaMaps' prior scoring stage and initialize the composition vector F uniformly across the >12,000 RefSeq database genomes. In the E-step, reweight each read's candidate-location posteriors by the current composition: P_r(i;F) = P_r(i)·(1/E_{r;g})·F_g. In the M-step, re-estimate F from the summed posteriors and recompute the total data likelihood. Iterate E- and M-steps until the likelihood converges; under the converged F, assign each read to its maximum-likelihood genome. Validate against the reported benchmark of r2 > 0.97 for sample composition estimation.

## Related tools

- **MetaMaps** (Implements the EM loop: computes composition-dependent posteriors P_r(i;F), re-estimates F in the M-step, and emits per-read maximum-likelihood assignments (./metamaps classify)) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Provides the minimizer-based approximate mapping algorithm that generates the candidate mapping locations the EM posteriors reweight) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify -t 5 --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Total data likelihood increases monotonically over EM iterations and convergence of the likelihood is used as the stopping criterion
- Estimated composition reaches the reported benchmark accuracy of r2 > 0.97 on simulated and real data when sample genomes or close relatives are in the database
- Per-read mapping qualities in classification_results.EM sum to 1 for each read
- EMFrequency and PotFrequency columns in classification_results.EM.WIMP are consistent with the read counts in the Absolute column after correction for unmapped reads
- Per-read assignments (classification_results.EM.reads2Taxon) agree with the converged maximum-likelihood mappings and species-level read assignment accuracy exceeds 94% on benchmarks

## Limitations

- Reads below the chosen minimum length get minimizer sets too small to determine mapping locations reliably and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- Closely related database genomes can only be distinguished by borrowing information across reads via the EM algorithm; database-sample mismatches (uncharacterized taxonomic entities) remain an unresolved concern requiring visual analysis or automatic thresholding

## Evidence

- [intro] Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm. This step also enables the disambiguation of: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm. This step also enables the disambiguation of"
- [intro] by using the EM algorithm for borrowing information across reads13,15, MetaMaps can distinguish between closely related database genomes: "by using the EM algorithm for borrowing information across reads13,15, MetaMaps can distinguish between closely related database genomes, a challenge that becomes more common as reference databases"
- [readme] The EM algorithm is carried out at this level.: [evidence span withheld — anchor status: not_found]
- [readme] The mapping qualities of all mappings for one read sum up to 1.: [evidence span withheld — anchor status: not_found]
- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
