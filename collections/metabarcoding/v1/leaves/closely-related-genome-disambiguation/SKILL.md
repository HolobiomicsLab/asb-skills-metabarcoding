---
name: closely-related-genome-disambiguation
description: Use when long metagenomic reads have multiple probabilistic mapping candidates
  across closely related genomes (e.g. within the >12,000-genome RefSeq miniSeq+H
  database) and per-read assignment plus sample composition estimates are both needed.
when_to_use_negative:
- Samples with a high proportion of short reads — reads below the minimum length end
  up with minimizer sets too small for reliable mapping, and MetaMaps does not address
  short-read-heavy samples
- Complex samples comprising many non-characterized entities absent or divergent from
  the reference database, where EM composition estimates cannot capture novel taxa
- K-mer-spectra-style taxonomic binning without per-read positional mapping information
  — MetaMaps' output is more information-rich but its EM stage requires per-read mapping-quality
  probabilities
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: 'Implements the EM algorithm: reads mappings produced by mapDirectly, estimates
    composition, and emits per-read maximum-likelihood assignments'
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Provides the minimizer-based approximate mapping algorithm that generates
    each read's candidate mapping locations feeding the EM stage
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:b4586292f56c311288c519bce6b99f6fbdadd4dd972f4ea92ba3c8e653fd6e16
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Closely-related genome disambiguation via EM-based sample composition estimation

## Summary

MetaMaps' second analysis stage runs an EM algorithm that estimates overall sample composition while simultaneously resolving each read's ambiguous candidate mapping locations to a maximum-likelihood genome, enabling strain-level discrimination between closely related database genomes.

## When to use

Use when long metagenomic reads have multiple probabilistic mapping candidates across closely related genomes (e.g. within the >12,000-genome RefSeq miniSeq+H database) and per-read assignment plus sample composition estimates are both needed.

## When NOT to use

- Samples with a high proportion of short reads — reads below the minimum length end up with minimizer sets too small for reliable mapping, and MetaMaps does not address short-read-heavy samples
- Complex samples comprising many non-characterized entities absent or divergent from the reference database, where EM composition estimates cannot capture novel taxa
- K-mer-spectra-style taxonomic binning without per-read positional mapping information — MetaMaps' output is more information-rich but its EM stage requires per-read mapping-quality probabilities

## Inputs

- Probabilistic mapping scores and mapping qualities for all classified long reads from MetaMaps' mapDirectly/scoring stage (classification_results mappings)
- MetaMaps classification database (e.g. miniSeq+H: >12,000 RefSeq microbial genomes plus the human reference genome)
- Long-read FASTQ input (PacBio/ONT), e.g. from the Zymo mock community sequencing runs

## Outputs

- classification_results.EM.WIMP: sample composition at taxonomic levels with Absolute, EMFrequency, and PotFrequency columns (EM carried out at 'definedGenomes' strain level)
- classification_results.EM.reads2Taxon: one line per read with read ID and maximum-likelihood taxon ID (x-prefixed internal IDs disambiguate genomes under one species node)
- classification_results.EM: final complete set of approximate read mappings with mapping qualities summing to 1 per read
- classification_results.EM.reads2Taxon.krona: Krona-format per-read assignments with quality values

## How to apply

After the approximate-mapping and probabilistic scoring stage, initialize the composition vector F uniformly across the database genomes and run EM: in each E-step compute composition-dependent posteriors P_r(i;F) = P_r(i)·(1/E_{r;g})·F_g per read over its candidate locations, reweighting mapping-quality-based probabilities by current composition; in each M-step re-estimate F from the summed posteriors and recompute the data likelihood. Iterate until likelihood convergence, so information is borrowed across reads to separate closely related genomes, then assign each read to its maximum-likelihood genome under the converged F. Run this via `metamaps classify`, which reports the EM composition in classification_results.EM.WIMP (with EMFrequency and PotFrequency columns, corrected for unmapped reads) and per-read assignments in classification_results.EM.reads2Taxon. Accept the composition estimate when it meets the benchmark accuracy of r2 > 0.97 for sample composition estimation, alongside >94% species-level read-assignment accuracy.

## Related tools

- **MetaMaps** (Implements the EM algorithm: reads mappings produced by mapDirectly, estimates composition, and emits per-read maximum-likelihood assignments) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Provides the minimizer-based approximate mapping algorithm that generates each read's candidate mapping locations feeding the EM stage) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- EM data likelihood converges across M-steps (likelihood convergence is the stopping criterion)
- Sample composition estimate meets the reported benchmark r2 > 0.97 against truth composition
- Species-level read-assignment accuracy reaches >94% on simulated and real data when sample genomes or close relatives are in the database
- Mapping qualities of all mappings for each read sum to 1 in classification_results.EM
- Every classified read appears exactly once in classification_results.EM.reads2Taxon, with EMFrequency + unmapped-read correction matching PotFrequency in the .WIMP file

## Limitations

- EM cannot account for sample-database incongruity: microbial reference databases cover only a fraction of microbial diversity, and HMP/CAMI experiments show visual analysis or automatic thresholding is needed to detect database-sample mismatches
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- With memory-efficient mapping (--maxmemory), actual memory usage fluctuates and can exceed the target

## Evidence

- [intro] Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm.: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [intro] EM borrowing disambiguates related genomes: "by using the EM algorithm for borrowing information across reads13,15, MetaMaps can distinguish between closely related database genomes"
- [readme] WIMP output definition: [evidence span withheld — anchor status: not_found]
- [readme] reads2Taxon output definition: [evidence span withheld — anchor status: not_found]
- [readme] Mapping quality normalization invariant: [evidence span withheld — anchor status: not_found]
