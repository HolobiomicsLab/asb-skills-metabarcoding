---
name: sequencing-error-rate-estimation-by-pass-number
description: Use when quantifying PacBio CCS read accuracy across circular-consensus
  pass numbers, or when deciding how many CCS passes are needed before full-length
  16S sequence variants can be trusted.
when_to_use_negative:
- Input reads are not PacBio CCS reads with pass-number information (e.g., short-read
  V4 amplicon data or non-CCS long reads) — the pass-number error model does not apply
- You already need downstream analysis of intragenomic 16S copy variants; the paper's
  filter step there focuses on substitutions and deliberately ignores insertions/deletions,
  so error-profile characterization is a prerequisite, not the analysis itself
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: PacBio
  role: Sequencing platform generating the circular-consensus (CCS) full-length 16S
    reads whose pass-number-dependent error profile is quantified
- name: Cross_match
  role: Aligner used to align CCS reads to the mock reference 16S genes (minscore
    750, penalty -9, masklevel 0); its parsed alignment output supplies the substitution/indel
    error calls
- name: ruffus (CGAT)
  role: Workflow framework in which the repository's analysis pipelines for each figure
    are implemented
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:90e94775c52449429e41ae66239ba9008d438755764c298e4fd01637596c5f08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus (CGAT)
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# sequencing-error-rate-estimation-by-pass-number

## Summary

Estimates the combined substitution, insertion, and deletion error frequency of PacBio circular-consensus (CCS) full-length 16S reads as a function of CCS pass number, using the mock community dataset and Cross_match alignments. Establishes that ten passes minimize the combined error rate to below 1.0% and reveals systematic deletion errors coincident with homopolymer runs.

## When to use

Use when quantifying PacBio CCS read accuracy across circular-consensus pass numbers, or when deciding how many CCS passes are needed before full-length 16S sequence variants can be trusted.

## When NOT to use

- Input reads are not PacBio CCS reads with pass-number information (e.g., short-read V4 amplicon data or non-CCS long reads) — the pass-number error model does not apply
- You already need downstream analysis of intragenomic 16S copy variants; the paper's filter step there focuses on substitutions and deliberately ignores insertions/deletions, so error-profile characterization is a prerequisite, not the analysis itself

## Inputs

- PacBio CCS FASTQ/FASTA read set from the 36-species mock community full-length 16S amplicons
- Mock reference 16S gene sequence set (per-species 16S references)
- Per-read circular-consensus pass number metadata for each CCS read

## Outputs

- Per-pass-number table of substitution, insertion, and deletion error counts and combined error frequencies
- Error-frequency versus pass-number curve (Supplementary Fig. 5 equivalent)
- Reference-position mapping of deletion errors with homopolymer-run (>=4 identical bases) overlap statistics (Supplementary Figs. 6-7 equivalents)

## How to apply

Download the mock community PacBio CCS read set and mock reference 16S gene sequences from the deposited repository github.com/TheJacksonLaboratory/weinstock_full_length_16s, then align CCS reads to the mock reference 16S genes with Cross_match using minscore 750, penalty -9, and masklevel 0. Parse the Cross_match alignments to tabulate substitution, insertion, and deletion errors per read, and group reads by their circular-consensus pass number. Compute the combined substitution/indel error frequency for each pass number and plot the error-frequency versus pass-number curve; accept the calibration when the combined rate falls below 1.0% at ten passes (reproducing Supplementary Fig. 5). As a second diagnostic, map each deletion error to its reference position and test overlap with homopolymer runs of >=4 identical consecutive bases in the 16S references, tabulating and visualizing the coincidence to reproduce Supplementary Figs. 6-7. Rationale: the paper found that sequencing errors in PacBio CCS reads, particularly deletions coincident with homopolymer runs, limit the ability to resolve highly similar sequences, so error profiles must be characterized before using full-length reads to discriminate intragenomic 16S copy variants.

## Related tools

- **PacBio** (Sequencing platform generating the circular-consensus (CCS) full-length 16S reads whose pass-number-dependent error profile is quantified)
- **Cross_match** (Aligner used to align CCS reads to the mock reference 16S genes (minscore 750, penalty -9, masklevel 0); its parsed alignment output supplies the substitution/indel error calls)
- **ruffus (CGAT)** (Workflow framework in which the repository's analysis pipelines for each figure are implemented) — https://github.com/cgat-developers/ruffus

## Examples

```
cross_match -minscore 750 -penalty -9 -masklevel 0 mock_ccs_reads.fasta mock_16s_references.fasta > ccs_vs_ref.align
```

## Evaluation signals

- Combined substitution/indel error frequency at ten CCS passes is below 1.0%, consistent with Supplementary Fig. 5
- Deletion errors show enrichment within homopolymer runs (>=4 identical consecutive bases) of the 16S references, as in Supplementary Figs. 6-7
- Cross_match was run with the exact parameters minscore 750, penalty -9, and masklevel 0
- Every error is attributed to a read with a defined circular-consensus pass number before aggregation
- Reproduction uses the mock community data and references from github.com/TheJacksonLaboratory/weinstock_full_length_16s and runs through the repository's ruffus pipelines

## Limitations

- Deletion errors coincident with homopolymer runs are systematic and may occur at a given frequency, hence may not be improved by greater sequencing effort
- The practice of ignoring insertions and deletions is imperfect, as demonstrated by a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes
- Future work is needed to determine how advances in sequencing platforms, chemistries, and computational approaches improve these errors

## Evidence

- [discussion] we highlighted the prevalence of sequencing errors in PacBio CCS reads as a factor that limits the ability to resolve highly similar sequences. A particular problem was deletion errors coincident: "we highlighted the prevalence of sequencing errors in PacBio CCS reads as a factor that limits the ability to resolve highly similar sequences. A particular problem was deletion errors coincident"
- [discussion] such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort: "such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [readme] This repository contains code used to perform the analysis reported in Johnson et al. 'The 16S rRNA gene for species and strain-level microbiome analysis': "This repository contains code used to perform the analysis reported in Johnson et al. 'The 16S rRNA gene for species and strain-level microbiome analysis'"
- [other] Align the CCS reads to the mock reference 16S gene set with Cross_match using minscore 750, penalty -9, and masklevel 0: [evidence span withheld — anchor status: not_found]
