---
name: em-based-sample-composition-inference
description: Use when you have long-read metagenomic data (FASTQ) and need simultaneous
  per-read taxonomic assignment and sample composition estimation, especially at strain
  level against a RefSeq-style database.
when_to_use_negative:
- 'Samples with a high proportion of short reads: reads below the minimum length threshold
  get minimizer sets too small to reliably determine mapping locations and remain
  unassigned'
- Complex samples comprising many non-characterized entities — database-sample incongruity
  (novel strains/species absent from the reference) is an explicit unresolved limitation
- Real-time/streaming analysis workflows — MetaMaps lacks streaming support needed
  for live clinical or sequencing-control applications
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Executes approximate mapping, probabilistic scoring, and EM-based estimation
    of overall sample composition on long reads
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying minimizer-based approximate mapping algorithm used by MetaMaps
    to generate candidate mapping locations
  repo: https://github.com/marbl/MashMap
- name: MEGAN
  role: 'Comparison baseline: LAST+MEGAN-LR per-read classifications are aggregated
    into taxon-level composition profiles for benchmarking'
- name: Bracken
  role: 'Comparison baseline: per-read classifications aggregated into composition
    profiles for r2/L1 evaluation'
- name: Centrifuge
  role: 'Comparison baseline: per-read classifications aggregated into composition
    profiles for r2/L1 evaluation'
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:51f9ee31112253a6a0c190a6bcfbf946ad5b0e9acd43f51599dd84cf08f52cf3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - Bracken
  - Centrifuge
  - LAST
  - MEGAN
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# EM-based sample composition inference

## Summary

Estimate taxonomic composition of a long-read (PacBio/ONT) metagenomic sample by scoring approximate mapping locations probabilistically and running an EM algorithm over per-read assignments, as implemented in MetaMaps. This disambiguates reads mapping to closely related strains and corrects for unmapped reads.

## When to use

Use when you have long-read metagenomic data (FASTQ) and need simultaneous per-read taxonomic assignment and sample composition estimation, especially at strain level against a RefSeq-style database.

## When NOT to use

- Samples with a high proportion of short reads: reads below the minimum length threshold get minimizer sets too small to reliably determine mapping locations and remain unassigned
- Complex samples comprising many non-characterized entities — database-sample incongruity (novel strains/species absent from the reference) is an explicit unresolved limitation
- Real-time/streaming analysis workflows — MetaMaps lacks streaming support needed for live clinical or sequencing-control applications

## Inputs

- Long-read FASTQ file (PacBio/ONT reads)
- MetaMaps classification database (e.g. miniSeq+H: >12,000 microbial genomes plus human reference, with taxonomy subdirectory)
- Ground-truth sample composition for benchmarking (strain level for i100/p25, species level for HMP7/Zymo/CAMI)

## Outputs

- classification_results.EM.WIMP — sample composition per taxonomic level with Absolute, EMFrequency, and PotFrequency columns
- classification_results.EM.reads2Taxon — per-read strain-level taxon assignments
- classification_results.EM.reads2Taxon.krona — per-read assignments in Krona format
- classification_results.EM.contigCoverage — per-1000bp-window read coverage
- composition_metrics.csv / composition_profiles.csv — Pearson r2 and L1-distance metrics and per-tool estimated compositions (benchmarking workflow)

## How to apply

Run MetaMaps' two-step workflow: first 'metamaps mapDirectly' generates candidate mapping locations using minimizer-based approximate mapping (built on MashMap), then 'metamaps classify' scores each location probabilistically and runs the EM algorithm at the 'definedGenomes' (strain) level to estimate sample composition. Read the 'classification_results.EM.WIMP' output, where 'EMFrequency' is the estimated frequency before correcting for unmapped reads and 'PotFrequency' is the final composition estimate, plus 'EM.reads2Taxon' for per-read strain-level taxon IDs (IDs prefixed with 'x' resolve via the database's extended taxonomy). Use '--maxmemory' (e.g. ~70% of RAM) and '-t' for multithreading. Compositions are validated against ground truth via Pearson r2 (0.88 at strain level on i100, 0.78 on p25, >0.99 at higher taxonomic levels) and L1 distance.

## Related tools

- **MetaMaps** (Executes approximate mapping, probabilistic scoring, and EM-based estimation of overall sample composition on long reads) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying minimizer-based approximate mapping algorithm used by MetaMaps to generate candidate mapping locations) — https://github.com/marbl/MashMap
- **MEGAN** (Comparison baseline: LAST+MEGAN-LR per-read classifications are aggregated into taxon-level composition profiles for benchmarking)
- **Bracken** (Comparison baseline: per-read classifications aggregated into composition profiles for r2/L1 evaluation)
- **Centrifuge** (Comparison baseline: per-read classifications aggregated into composition profiles for r2/L1 evaluation)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- classification_results.EM.WIMP exists and contains the expected columns (Absolute, EMFrequency, PotFrequency); EM is run at the 'definedGenomes' strain-resolved level
- Mapping qualities of all mappings for a single read sum to 1 in classification_results.EM
- Reproduced Pearson r2 between estimated and true abundances falls near reported values: 0.88 (i100) and 0.78 (p25) at strain level; >0.99 at higher taxonomic levels; 0.98 (HMP7) and 0.97 (Zymo) at species level
- Per-read assignments in EM.reads2Taxon resolve 'x'-prefixed internal taxon IDs against the database's extended taxonomy directory without unmapped IDs
- L1 distance between estimated and ground-truth composition is computed for every tool-dataset combination and matches reported Fig. 2 / Fig. 5 / Supplementary Data 3 patterns

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed
- Complex samples comprising many non-characterized entities remain challenging; likelihood of sample-database mismatches depends on the source environment
- No explicit term for genomic divergence is integrated into the statistical models, limiting explicit detection of novel strains and species
- Memory usage with '--maxmemory' fluctuates and may exceed the heuristic target

## Evidence

- [intro] Two-stage scoring and EM composition estimation: "Second, each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm."
- [abstract] Abstract integration claim: "Integrating approximate mapping with probabilistic scoring and EM-based estimation of sample composition"
- [readme] README description of simultaneous output: "It simultaenously carries out read assignment and sample composition estimation."
- [readme] README WIMP output semantics: [evidence span withheld — anchor status: not_found]
- [abstract] Reported accuracy headline: "MetaMaps achieves >94% accuracy for species-level read assignment and r2 > 0.97 for the estimation of sample composition on both simulated and real data"
