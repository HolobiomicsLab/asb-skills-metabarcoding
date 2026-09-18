---
name: alignment-identity-distribution-analysis
description: Use when long-read metagenomic reads map to a reference database but
  identity or coverage patterns suggest the sample genome is absent or diverged from
  the database, e.g. HMP7 Actinomyces meyeri or CAMI samples.
when_to_use_negative:
- Sample genomes or close relatives are confidently in the database and identities
  cluster at the expected ~0.88 mode, so no mismatch analysis is needed
- Sample has a high proportion of short reads, since reads below the minimum length
  end up with minimizer sets too small to reliably determine mapping locations
- Precise base-to-base alignments are required; MetaMaps estimates approximate alignment
  identities (Poisson/binomial approximations) rather than computing explicit alignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Produces the per-read mapping locations, estimated alignment identities, mapping
    qualities, and per-window coverage used to build the identity distribution and
    coverage plots
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying minimizer/minhash-based approximate mapping algorithm on which
    MetaMaps' identity estimation is built
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:6aaa6de6e2c0a7a52677c9a98d562625f6681129196bd820d832de8cd9ae999b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# alignment-identity-distribution-analysis

## Summary

Uses MetaMaps' per-read mapping output (mapping locations, estimated alignment identities, and coverage per 1000 bp window) to detect database-sample mismatches, e.g. an in-sample genome whose closest reference is divergent, by visualizing uneven read-depth coverage along the reference and a shift in the alignment-identity distribution away from its expected mode (~0.88 for correct references; ~82% median identity signals mismatch on CAMI).

## When to use

Use when long-read metagenomic reads map to a reference database but identity or coverage patterns suggest the sample genome is absent or diverged from the database, e.g. HMP7 Actinomyces meyeri or CAMI samples.

## When NOT to use

- Sample genomes or close relatives are confidently in the database and identities cluster at the expected ~0.88 mode, so no mismatch analysis is needed
- Sample has a high proportion of short reads, since reads below the minimum length end up with minimizer sets too small to reliably determine mapping locations
- Precise base-to-base alignments are required; MetaMaps estimates approximate alignment identities (Poisson/binomial approximations) rather than computing explicit alignments

## Inputs

- Long-read FASTQ metagenomic sample (e.g. HMP7 Actinomyces reads or CAMI sample)
- MetaMaps classification database (e.g. miniSeq+H / comprehensive RefSeq with >12,000 genomes)
- MetaMaps mapping output file (classification_results.EM)

## Outputs

- Per-read estimated alignment identities and read lengths (classification_results.EM.lengthAndIdentitiesPerMappingUnit)
- Per-1000bp-window read-depth coverage along mapped contigs (classification_results.EM.contigCoverage)
- Coverage-along-reference plot showing uneven spatial coverage (e.g. Actinomyces meyeri)
- Alignment-identity distribution plot showing shift away from the ~0.88 mode (median ~82% on CAMI mismatch)
- Summary statistics supporting a database-sample mismatch detection call

## How to apply

Run MetaMaps on the long-read sample (e.g. HMP7 Actinomyces reads) against the comprehensive RefSeq database using 'metamaps mapDirectly' followed by 'metamaps classify'. From the output file 'classification_results.EM.lengthAndIdentitiesPerMappingUnit' extract the per-read estimated alignment identities and read lengths, and from 'classification_results.EM.contigCoverage' extract per-window (1000 bp) read coverage for the mapped contig. Visualize read-depth coverage along the reference contig (e.g. Actinomyces meyeri) to reveal uneven spatial coverage, and plot the alignment-identity distribution to show a shift away from the ~0.88 identity mode expected when the correct genome is in the database. On the CAMI sample, summarize identity statistics and confirm the ~82% median-identity mismatch signal; both visual and automatic thresholding of these patterns constitute evidence of database-sample incongruity.

## Related tools

- **MetaMaps** (Produces the per-read mapping locations, estimated alignment identities, mapping qualities, and per-window coverage used to build the identity distribution and coverage plots) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying minimizer/minhash-based approximate mapping algorithm on which MetaMaps' identity estimation is built) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results && ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Alignment-identity distribution for a correctly represented genome peaks near the ~0.88 mode; a shifted distribution (e.g. ~82% median identity on CAMI) flags a database-sample mismatch
- Coverage plot along the reference contig shows uneven spatial coverage characteristic of a divergent/missing genome rather than uniform coverage
- classification_results.EM.lengthAndIdentitiesPerMappingUnit contains Identity and Length fields for every classified read, sorted by mapped contig
- classification_results.EM.contigCoverage windows report nBases/readCoverage for 1000 bp windows, matching the expected schema (taxonID, equalCoverageUnitLabel, contigID, start, stop)
- Leaving out-of-database reads unclassified (e2 eukaryotic contaminants) shows >99% precision/recall, confirming the identity-based mismatch signal is not driven by false classification

## Limitations

- The evidenceUnknownSpecies output statistics are 'not particularly useful in their current state'; visual inspection of coverage and identity patterns should take precedence
- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations, so they contribute no identities
- Detection of novelty from long-read data lacks an explicit genomic-divergence term in the statistical models, limiting explicit detection of novel strains/species

## Evidence

- [abstract] MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities between sample and reference genomes: "MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities between sample and reference genomes"
- [discussion] HMP7/CAMI visual analysis for mismatch detection: "As we have shown in the HMP7 and CAMI experiments, visual analysis or automatic thresholding can contribute to detecting and dealing with database-sample mismatches."
- [readme] README description of lengthAndIdentitiesPerMappingUnit output: [evidence span withheld — anchor status: not_found]
- [readme] README description of contigCoverage output: [evidence span withheld — anchor status: not_found]
- [readme] README caveat on evidenceUnknownSpecies statistics: [evidence span withheld — anchor status: not_found]
