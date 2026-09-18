---
name: mock-community-reference-mapping
description: Use when PacBio CCS (or similar long-read) amplicon reads from a defined
  mock community need to be error-profiled against known reference 16S gene sequences,
  or when error frequency must be stratified by CCS pass number.
when_to_use_negative:
- Input reads have no matched reference truth set (e.g. in-vivo community samples)
  — error frequencies cannot be computed against known mock references
- The goal is taxonomic classification or OTU clustering of full-length 16S reads
  rather than sequencing-error characterization
- Analysis targets Illumina short-read variable-region data, where the error profile
  and pass-number concept do not apply
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3697
tools:
- name: Cross_match
  role: Aligns CCS reads to the mock reference 16S gene set (minscore 750, penalty
    -9, masklevel 0); alignment output is parsed to tabulate per-read substitution/indel
    errors
- name: PacBio
  role: Platform generating the CCS reads whose error profile versus circular-consensus
    pass number is being characterized
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the analysis pipelines in
    the weinstock_full_length_16s repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:db4040dd4a96adcdb5917304ceebe047ccdfa8127ee406a8ef85ef9bacf664c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# mock-community-reference-mapping

## Summary

Aligns PacBio CCS full-length 16S reads from a mock community against the known mock reference 16S gene sequences with Cross_match (minscore 750, penalty -9, masklevel 0) so per-read substitution, insertion, and deletion errors can be tabulated against reference truth, e.g. grouped by circular-consensus pass number.

## When to use

Use when PacBio CCS (or similar long-read) amplicon reads from a defined mock community need to be error-profiled against known reference 16S gene sequences, or when error frequency must be stratified by CCS pass number.

## When NOT to use

- Input reads have no matched reference truth set (e.g. in-vivo community samples) — error frequencies cannot be computed against known mock references
- The goal is taxonomic classification or OTU clustering of full-length 16S reads rather than sequencing-error characterization
- Analysis targets Illumina short-read variable-region data, where the error profile and pass-number concept do not apply

## Inputs

- PacBio CCS read set (FASTA/FASTQ) from the 36-species mock community full-length 16S amplicons
- Mock reference 16S gene sequences from github.com/TheJacksonLaboratory/weinstock_full_length_16s
- Per-read circular-consensus pass-number metadata for grouping reads

## Outputs

- Per-read tabulation of substitution, insertion, and deletion errors parsed from Cross_match alignments, grouped by pass number
- Combined substitution/indel error-frequency versus pass-number curve (reproducing Supplementary Fig. 5)
- Reference-position mapping of deletion errors with homopolymer-run (>=4 identical consecutive bases) overlap statistics (Supplementary Figs. 6-7)

## How to apply

Download the mock community PacBio CCS read set and the mock reference 16S gene sequences from the deposited repository github.com/TheJacksonLaboratory/weinstock_full_length_16s. Align the CCS reads to the mock reference 16S gene set with Cross_match using minscore 750, penalty -9, and masklevel 0. Parse the Cross_match alignments to tabulate substitution, insertion, and deletion errors per read, grouping reads by their circular-consensus pass number. Compute the combined substitution/indel error frequency per pass number and plot the error-frequency versus pass-number curve; downstream, map each deletion to its reference position and test overlap with homopolymer runs (>=4 identical consecutive bases) in the reference sequences.

## Related tools

- **Cross_match** (Aligns CCS reads to the mock reference 16S gene set (minscore 750, penalty -9, masklevel 0); alignment output is parsed to tabulate per-read substitution/indel errors)
- **PacBio** (Platform generating the CCS reads whose error profile versus circular-consensus pass number is being characterized)
- **ruffus** (CGAT ruffus pipeline framework used to organize the analysis pipelines in the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus

## Examples

```
cross_match mock_ccs_reads.fasta mock_16s_references.fasta -minscore 750 -penalty -9 -masklevel 0
```

## Evaluation signals

- Accept the reproduction when the combined substitution/indel error frequency at ten passes is below 1.0%, consistent with Supplementary Fig. 5
- Error-frequency versus pass-number curve decreases monotonically toward the minimum around ten passes
- Deletion errors show enrichment within homopolymer runs (>=4 identical consecutive bases) in the 16S references, as in Supplementary Figs. 6-7
- Every deletion error maps to a valid reference position in the mock 16S gene set
- Cross_match run parameters match minscore 750, penalty -9, and masklevel 0

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences
- Such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort
- The paper chose to focus on substitutions and ignore insertions/deletions for intragenomic 16S copy polymorphism analysis because of these platform-specific errors; a single deletion in one of the seven E. coli K-12 MG1655 16S genes demonstrates that ignoring indels is an imperfect approach

## Evidence

- [supplementary] Cross_match alignment parameters: [evidence span withheld — anchor status: not_found]
- [supplementary] Validation criterion at ten passes: [evidence span withheld — anchor status: not_found]
- [supplementary] Finding on minimum error rate: [evidence span withheld — anchor status: not_found]
- [readme] Repository provenance: "This repository contains code used to perform the analysis reported in Johnson et al. 'The 16S rRNA gene for species and strain-level microbiome analysis'"
- [discussion] Deletion errors coincide with homopolymer runs: "A particular problem was deletion errors coincident"
