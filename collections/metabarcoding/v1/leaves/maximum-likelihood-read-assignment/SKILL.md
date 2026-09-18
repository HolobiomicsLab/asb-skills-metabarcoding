---
name: maximum-likelihood-read-assignment
description: Use when long reads have probabilistic mapping scores to multiple candidate
  (possibly strain-level) reference genomes and each read must be resolved to a single
  taxon while jointly estimating sample composition.
when_to_use_negative:
- 'Samples with a high proportion of short reads: reads below the minimum length produce
  minimizer sets too small for reliable mapping and remain unassigned'
- Complex samples comprising many non-characterized entities not represented in the
  RefSeq database, where database-sample mismatch dominates
- When only raw k-mer-based taxonomic labels without positional/mapping-quality information
  are sufficient — this skill is specific to mapping-based long-read assignment
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaMaps
  role: Implements the probabilistic scoring and EM-based classification that assigns
    each read to its maximum-likelihood genome and estimates sample composition
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Provides the minimizer-based approximate mapping algorithm that generates
    candidate mapping locations for each read prior to EM assignment
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:2271a9cc478b4ad761c3f97e8f354739c15b60ae383e5fc99145492e3542a233
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# maximum-likelihood-read-assignment

## Summary

Resolve ambiguous long-read mapping locations by assigning each read to its maximum-likelihood genome under an EM-estimated sample composition, as implemented in MetaMaps' classify step. This step simultaneously disambiguates reads shared between closely related RefSeq genomes and yields per-read taxonomic assignments plus composition estimates.

## When to use

Use when long reads have probabilistic mapping scores to multiple candidate (possibly strain-level) reference genomes and each read must be resolved to a single taxon while jointly estimating sample composition.

## When NOT to use

- Samples with a high proportion of short reads: reads below the minimum length produce minimizer sets too small for reliable mapping and remain unassigned
- Complex samples comprising many non-characterized entities not represented in the RefSeq database, where database-sample mismatch dominates
- When only raw k-mer-based taxonomic labels without positional/mapping-quality information are sufficient — this skill is specific to mapping-based long-read assignment

## Inputs

- Mappings file from MetaMaps approximate mapping (e.g. classification_results from 'metamaps mapDirectly')
- Classification database directory (e.g. miniSeq+H, >12,000 RefSeq microbial genomes plus human reference, incl. extended taxonomy)
- Long-read FASTQ (PacBio/Oxford Nanopore) input to the mapping step

## Outputs

- classification_results.EM.WIMP: sample composition at multiple taxonomic levels (Absolute, EMFrequency, PotFrequency)
- classification_results.EM.reads2Taxon: one line per read with read ID and maximum-likelihood taxon ID
- classification_results.EM.reads2Taxon.krona: Krona-format per-read assignments
- classification_results.EM: complete set of approximate mappings with estimated identity and mapping qualities summing to 1 per read
- classification_results.EM.contigCoverage and classification_results.EM.lengthAndIdentitiesPerMappingUnit

## How to apply

Run MetaMaps' classification stage on the mappings produced by the minimizer-based approximate mapping step ('metamaps mapDirectly'). The EM algorithm is carried out at the 'definedGenomes' (strain) level: it iteratively re-estimates the composition vector while reweighting each read's mapping-quality-based probabilities over candidate locations, borrowing information across reads to disambiguate closely related database genomes. Under the converged composition, each read is assigned to its maximum-likelihood genome. Outputs include 'classification_results.EM.WIMP' (composition: Absolute, EMFrequency, PotFrequency columns), 'classification_results.EM.reads2Taxon' (read ID to taxon ID, where 'x'-prefixed IDs are MetaMaps-internal strain-disambiguating taxa), and 'classification_results.EM' with mapping qualities that sum to 1 per read. Correct application is judged by benchmark performance: >94% species-level read accuracy and r2 > 0.97 for composition estimation when sample genomes or close relatives are in the database.

## Related tools

- **MetaMaps** (Implements the probabilistic scoring and EM-based classification that assigns each read to its maximum-likelihood genome and estimates sample composition) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Provides the minimizer-based approximate mapping algorithm that generates candidate mapping locations for each read prior to EM assignment) — https://github.com/marbl/MashMap

## Examples

```
./metamaps classify -t 5 --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Mapping qualities of all mappings for one read in classification_results.EM sum to 1
- Benchmark accuracy meets >94% species-level read assignment and r2 > 0.97 for sample composition estimation
- EM likelihood convergence is the stopping criterion; WIMP PotFrequency values account for unmapped reads after EMFrequency
- Reads2Taxon lines cover all classified reads; 'x'-prefixed taxon IDs resolve to the extended database taxonomy (taxonomy sub-directory of the database)

## Limitations

- EM assignment cannot detect novel strains/species explicitly — no explicit term for genomic divergence is integrated into the statistical models
- Database-sample incongruity (uncharacterized taxonomic entities) can bias assignments; visual analysis or automatic thresholding of coverage/identity is needed to detect mismatches (as shown in HMP and CAMI experiments)
- Robustness against large contaminant genomes is claimed as an advantage over k-mer methods, but large contaminant reference genomes must still be present in the database to be handled

## Evidence

- [intro] each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm. This step also enables the disambiguation of: "each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm. This step also enables the disambiguation of"
- [intro] by using the EM algorithm for borrowing information across reads13,15, MetaMaps can distinguish between closely related database genomes: "by using the EM algorithm for borrowing information across reads13,15, MetaMaps can distinguish between closely related database genomes, a challenge that becomes more common as reference databases"
- [readme] The EM algorithm is carried out at this level.: [evidence span withheld — anchor status: not_found]
- [readme] The mapping qualities of all mappings for one read sum up to 1.: [evidence span withheld — anchor status: not_found]
- [abstract] MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
