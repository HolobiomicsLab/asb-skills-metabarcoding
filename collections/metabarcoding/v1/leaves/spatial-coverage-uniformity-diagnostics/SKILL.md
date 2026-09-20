---
name: spatial-coverage-uniformity-diagnostics
description: Use when a long-read (PacBio/ONT) metagenomic sample has been classified
  against a RefSeq-scale database and you need to flag reference genomes that are
  divergent from, or absent from, the database via coverage and identity patterns.
when_to_use_negative:
- The sample has a high proportion of short reads — reads below the minimum length
  end up with minimizer sets too small for reliable mapping, so coverage/identity
  diagnostics will be sparse or unreliable
- You need automated novelty detection with an explicit genomic-divergence term —
  MetaMaps' statistical models do not integrate such a term and detection of novelty
  from long reads remains an open question
- The goal is purely compositional estimation (WIMP frequencies) rather than incongruity
  detection — use classification_results.EM.WIMP directly instead
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3168
tools:
- name: MetaMaps
  role: Produces the per-read mapping locations, estimated alignment identities, and
    1000-bp-window coverage tables that feed the uniformity diagnostics
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Minimizer/minmer-based approximate mapping algorithm underlying MetaMaps'
    mapping stage
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:b8cae74dd5a410165b6b01538884162b3c26a7a077387f576c5c59c917bacdcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Spatial coverage uniformity diagnostics

## Summary

Detects database-sample mismatches in long-read metagenomic assignments by inspecting MetaMaps' per-window read-depth coverage and estimated alignment-identity distributions — uneven spatial coverage plus identity depression (e.g. median identity ~82% on CAMI, mode shifted away from ~0.88 on HMP7) signals that the true sample genome diverges from the assigned reference.

## When to use

Use when a long-read (PacBio/ONT) metagenomic sample has been classified against a RefSeq-scale database and you need to flag reference genomes that are divergent from, or absent from, the database via coverage and identity patterns.

## When NOT to use

- The sample has a high proportion of short reads — reads below the minimum length end up with minimizer sets too small for reliable mapping, so coverage/identity diagnostics will be sparse or unreliable
- You need automated novelty detection with an explicit genomic-divergence term — MetaMaps' statistical models do not integrate such a term and detection of novelty from long reads remains an open question
- The goal is purely compositional estimation (WIMP frequencies) rather than incongruity detection — use classification_results.EM.WIMP directly instead

## Inputs

- Long-read FASTQ (PacBio/Oxford Nanopore) metagenomic read set
- MetaMaps classification database (e.g. miniSeq+H with >12,000 microbial genomes plus the human reference)
- MetaMaps output files: classification_results.EM, classification_results.EM.contigCoverage, classification_results.EM.lengthAndIdentitiesPerMappingUnit, classification_results.EM.reads2Taxon

## Outputs

- Read-depth coverage profile along each assigned reference contig in 1000 bp windows
- Per-read estimated alignment-identity distribution (e.g. mode/median identity per genome)
- Binary or ranked flag of database-sample mismatch per assigned genome (uneven coverage + identity depression)

## How to apply

Run MetaMaps in two steps (mapDirectly then classify) on the long-read FASTQ against the miniSeq+H/comprehensive RefSeq database to obtain per-read mappings, estimated alignment identities, and mapping qualities. Extract per-read mapping locations and estimated alignment identities from the .EM mapping and .EM.lengthAndIdentitiesPerMappingUnit outputs for the genome(s) of interest. Visualize read-depth coverage along the assigned reference (e.g. Actinomyces meyeri in the HMP7 sample) using the .EM.contigCoverage windows of 1000 base pairs to reveal spatially uneven coverage. Plot the alignment-identity distribution and look for a mode shift away from the expected ~0.88 identity mode (HMP7) or a depressed ~82% median identity (CAMI) as evidence of database-sample mismatch. Judge success when visual analysis or automatic thresholding of these coverage and identity patterns correctly flags genomes whose true source is a close relative rather than the assigned reference.

## Related tools

- **MetaMaps** (Produces the per-read mapping locations, estimated alignment identities, and 1000-bp-window coverage tables that feed the uniformity diagnostics) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Minimizer/minmer-based approximate mapping algorithm underlying MetaMaps' mapping stage) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- classification_results.EM.contigCoverage is populated with 1000 bp windows per contig, and flagged genomes show spatially uneven read coverage along the reference (e.g. Actinomyces meyeri in HMP7)
- The alignment-identity distribution for the HMP7 Actinomyces reads shows a mode shifted away from the ~0.88 identity mode
- Median estimated alignment identity on the CAMI sample is ~82%, reproducing the mismatch signal of Supplementary Data 4
- Per-read identities and mapping qualities are available for all classified reads (mapping qualities across a read's mappings sum to 1)
- On e2-type out-of-database contamination, precision/recall of leaving contaminant reads unclassified exceeds 99% rather than producing false assignments whose coverage would look uniform

## Limitations

- classification_results.EM.evidenceUnknownSpecies statistics are 'not particularly useful in their current state; visual inspection of coverage and identity patterns should take precedence'
- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned, thinning coverage
- Complex samples comprising many non-characterized entities remain challenging; microbial reference databases comprise only a fraction of total microbial genome diversity

## Evidence

- [abstract] MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities between sample and reference genomes: "MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities between sample and reference genomes"
- [discussion] HMP7/CAMI visual analysis quote: "As we have shown in the HMP7 and CAMI experiments, visual analysis or automatic thresholding can contribute to detecting and dealing with database-sample mismatches."
- [readme] contigCoverage output description: [evidence span withheld — anchor status: not_found]
- [readme] identity per read output description: [evidence span withheld — anchor status: not_found]
- [readme] evidenceUnknownSpecies caveat: [evidence span withheld — anchor status: not_found]
