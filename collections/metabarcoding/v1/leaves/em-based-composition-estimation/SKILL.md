---
name: em-based-composition-estimation
description: Use when long reads (PacBio/ONT) have already been approximately mapped
  and scored by MetaMaps and you need an overall sample composition estimate plus
  disambiguation of reads with multiple candidate mapping locations.
when_to_use_negative:
- Sample contains a high proportion of short reads — reads below the minimum length
  get minimizer sets too small to reliably determine mapping locations and end up
  unassigned
- The sample comprises many non-characterized entities absent from the reference database
  — database-sample incongruity (e.g. HMP7/CAMI mismatches) undermines composition
  estimation
- Input is raw reads without prior MetaMaps approximate mapping and probabilistic
  scoring — this EM stage requires mapping-quality-based probabilities as input
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: MetaMaps
  role: Implements the EM-based sample composition estimation and composition-dependent
    read reassignment (metamaps classify)
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Minimizer-based approximate mapping algorithm underlying the candidate mapping
    locations that feed the EM stage
  repo: https://github.com/marbl/MashMap
merged_aliases:
- em-based-genome-composition-estimation
merged_alias_records:
- alias: em-based-genome-composition-estimation
  slug: em-based-genome-composition-estimation
  jaccard_score: 0.8
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:57e5ec4b7b8689df1cf540e04cd0516c208dfc06cff43430843dddc03e566129
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Kraken
  - Kraken 2
  - MEGAN
  - MashMap
  - MetaMaps
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# EM-based composition estimation

## Summary

MetaMaps' second analysis stage: probabilistic per-read mapping scores from a >12,000-genome RefSeq database are iteratively reweighted via the EM algorithm to estimate sample composition and resolve each read to a maximum-likelihood genome, giving strain-level (definedGenomes) resolution.

## When to use

Use when long reads (PacBio/ONT) have already been approximately mapped and scored by MetaMaps and you need an overall sample composition estimate plus disambiguation of reads with multiple candidate mapping locations.

## When NOT to use

- Sample contains a high proportion of short reads — reads below the minimum length get minimizer sets too small to reliably determine mapping locations and end up unassigned
- The sample comprises many non-characterized entities absent from the reference database — database-sample incongruity (e.g. HMP7/CAMI mismatches) undermines composition estimation
- Input is raw reads without prior MetaMaps approximate mapping and probabilistic scoring — this EM stage requires mapping-quality-based probabilities as input

## Inputs

- Probabilistic mapping locations, estimated alignment identities, and mapping qualities for all classified long reads (output of metamaps mapDirectly / the scoring stage)
- MetaMaps classification database (e.g. miniSeq+H: >12,000 microbial genomes plus the human reference, with taxonomy sub-directory)
- Long-read FASTQ metagenomic dataset (e.g. i100, p25, CAMI, HMP7, Zymo mock community reads)

## Outputs

- classification_results.EM.WIMP: sample composition per taxonomic level with Absolute, EMFrequency, and PotFrequency columns (level 'definedGenomes' = strain-level resolution, where the EM is carried out)
- classification_results.EM.reads2Taxon: one line per read mapping read ID to the assigned taxon/genome (maximum-likelihood assignment)
- classification_results.EM.reads2Taxon.krona: per-read assignments in Krona format
- Converged sample composition vector (validated against r2 > 0.97 benchmark)

## How to apply

Load the probabilistic mapping scores and mapping qualities produced by the MetaMaps mapping/scoring stage and initialize the composition vector F uniformly across the database genomes. In the E-step, compute composition-dependent posteriors P_r(i;F) = P_r(i)·(1/E_{r;g})·F_g for each read over its candidate mapping locations; in the M-step, re-estimate F from the summed posteriors and recompute the total data likelihood. Iterate E- and M-steps until the likelihood converges — convergence is the stopping criterion — letting information be borrowed across reads to distinguish closely related genomes. Under the converged composition, assign each read to its maximum-likelihood genome. Run this as the `metamaps classify` command, which emits the EM.WIMP composition table (with EMFrequency before and PotFrequency after unmapped-read correction) and the per-read reads2Taxon assignments. Accept the composition estimate only if it meets the reported benchmark accuracy of r2 > 0.97.

## Related tools

- **MetaMaps** (Implements the EM-based sample composition estimation and composition-dependent read reassignment (metamaps classify)) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Minimizer-based approximate mapping algorithm underlying the candidate mapping locations that feed the EM stage) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- EM data likelihood increases monotonically and converges — convergence is the explicit stopping criterion
- Estimated composition matches the reported benchmark accuracy of r2 > 0.97 for sample composition estimation on simulated and real data
- EM.WIMP file contains the expected columns (Absolute, EMFrequency, PotFrequency) with 'definedGenomes' as the strain-level row level
- Per-read mapping qualities of all mappings for one read sum to 1 before EM reassignment
- reads2Taxon assigns every classified read exactly one taxon ID (x-prefixed IDs resolvable via the database's taxonomy sub-directory)

## Limitations

- Reads that fall below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- Database-sample mismatches (uncharacterized taxonomic entities) require visual analysis or automatic thresholding to detect, as shown in the HMP7 and CAMI experiments

## Evidence

- [intro] Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm.: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
- [readme] The level "definedGenomes" represents strain-level resolution (i.e., the defined genomes in the classification database). The EM algorithm is carried out at this level.: [evidence span withheld — anchor status: not_found]
- [readme] The mapping qualities of all mappings for one read sum up to 1.: [evidence span withheld — anchor status: not_found]
