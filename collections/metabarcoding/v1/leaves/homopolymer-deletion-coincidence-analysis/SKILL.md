---
name: homopolymer-deletion-coincidence-analysis
description: Use when analyzing PacBio CCS read-to-reference alignments of full-length
  16S amplicons to quantify whether deletion errors coincide with homopolymer runs,
  or to explain why highly similar 16S copies cannot be resolved.
when_to_use_negative:
- Input reads are Illumina short-read data of 16S variable regions rather than PacBio
  CCS full-length amplicons
- The analysis goal is substitution-based discrimination of intragenomic 16S copies
  where indels are deliberately ignored (the article's filter step focuses on substitutions
  and ignores insertions/deletions)
- No reference 16S sequences are available, so deletion positions cannot be mapped
  to homopolymer context
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: Cross_match
  role: Aligns PacBio CCS reads to mock reference 16S gene set (minscore 750, penalty
    -9, masklevel 0) from which deletion errors and their reference positions are
    parsed
- name: PacBio
  role: Sequencing platform whose CCS reads supply the deletion errors being characterized
- name: ruffus (CGAT)
  role: Pipeline framework used to organize the analysis code per figure in the deposited
    repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:e4e2f66bb6795b00dc6d93b74fa605b1a4e4d75d1437f929a01680f565c98f0c
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
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# homopolymer-deletion-coincidence-analysis

## Summary

Characterizes systematic deletion errors in PacBio CCS full-length 16S reads by mapping every alignment deletion to its reference position and testing overlap with homopolymer runs (>=4 identical consecutive bases) in the 16S reference sequences, as in Supplementary Figs. 6-7 of Johnson et al. 2019.

## When to use

Use when analyzing PacBio CCS read-to-reference alignments of full-length 16S amplicons to quantify whether deletion errors coincide with homopolymer runs, or to explain why highly similar 16S copies cannot be resolved.

## When NOT to use

- Input reads are Illumina short-read data of 16S variable regions rather than PacBio CCS full-length amplicons
- The analysis goal is substitution-based discrimination of intragenomic 16S copies where indels are deliberately ignored (the article's filter step focuses on substitutions and ignores insertions/deletions)
- No reference 16S sequences are available, so deletion positions cannot be mapped to homopolymer context

## Inputs

- PacBio CCS full-length 16S amplicon reads (FASTA/FASTQ) from the 36-species mock community
- Mock community reference 16S gene sequences (from github.com/TheJacksonLaboratory/weinstock_full_length_16s)
- Cross_match alignment output (parsed per-read error records with reference positions)

## Outputs

- Per-read deletion error list with mapped reference positions
- Deletion-to-homopolymer-run overlap table (coincidence counts)
- Visualization of deletion error coincidence with homopolymer runs (Supplementary Figs. 6-7 style plots)

## How to apply

First align the PacBio CCS reads to the mock reference 16S gene set with Cross_match using minscore 750, penalty -9, and masklevel 0, then parse the alignments to tabulate substitution, insertion, and deletion errors per read. Next, extract each deletion error's position in the reference sequence and annotate the reference for homopolymer runs of >=4 identical consecutive bases. Test each deletion's overlap with these runs and tabulate/visualize the coincidence to reproduce Supplementary Figs. 6-7. The rationale is that PacBio CCS deletions are systematic and homopolymer-associated, meaning such errors occur at a given frequency and may not be improved by greater sequencing effort — a key reason substitutions, not indels, were used for intragenomic 16S copy discrimination.

## Related tools

- **Cross_match** (Aligns PacBio CCS reads to mock reference 16S gene set (minscore 750, penalty -9, masklevel 0) from which deletion errors and their reference positions are parsed)
- **PacBio** (Sequencing platform whose CCS reads supply the deletion errors being characterized)
- **ruffus (CGAT)** (Pipeline framework used to organize the analysis code per figure in the deposited repository) — https://github.com/cgat-developers/ruffus

## Examples

```
cross_match ccs_reads.fasta mock_16s_refs.fasta -minscore 750 -penalty -9 -masklevel 0
```

## Evaluation signals

- Combined substitution/indel error frequency at ten CCS passes falls below 1.0%, consistent with Supplementary Fig. 5
- Deletion errors show enrichment within homopolymer runs (>=4 identical consecutive bases) in the 16S references, reproducing Supplementary Figs. 6-7
- Every parsed deletion in the Cross_match alignments maps to a valid reference position in the mock 16S gene set
- Per-pass grouping of errors reproduces the error-frequency versus pass-number curve with a minimum around ten passes

## Limitations

- Deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences, so the approach cannot distinguish copies differing only by indels
- Such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort
- The article notes that ignoring insertions/deletions is imperfect — a single deletion was observed in one of the seven E. coli strain K-12 substr. MG1655 16S genes
- Future advances in sequencing platforms, chemistries, and computational approaches would be needed to improve these errors

## Evidence

- [discussion] Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences: "we highlighted the prevalence of sequencing errors in PacBio CCS reads as a factor that limits the ability to resolve highly similar sequences. A particular problem was deletion errors coincident"
- [discussion] Systematic errors occur at a fixed frequency not improved by more sequencing: "such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort"
- [other] Workflow: map deletions to homopolymer runs >=4 bases: [evidence span withheld — anchor status: not_found]
- [readme] Repository organizes analysis as ruffus pipelines per figure: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
