---
name: indel-aware-alignment-postprocessing
description: Use when aligning PacBio CCS full-length 16S or Illumina WGS reads to
  a single 16S reference gene and per-position substitution profiles of intragenomic
  16S copies are needed, but raw indel tallies would be contaminated by platform-specific
  errors.
when_to_use_negative:
- Indel polymorphisms are the analysis target (e.g., validating a real copy-specific
  deletion) — this skill discards insertions and deletions by design
- Input reads are short amplicon reads of 16S variable regions (e.g., V4) rather than
  full-length ~1500 bp reads; intragenomic copy substitution profiles are not resolvable
  from sub-regions
- The goal is community-wide OTU clustering or taxonomic profiling rather than characterizing
  intragenomic 16S copy variation against a known reference genome
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0637
tools:
- name: minimap2
  role: Global aligner option named in the workflow for mapping PacBio CCS full-length
    16S reads to the single E. coli K-12 MG1655 16S reference before substitution
    tallying
- name: parasail
  role: Smith-Waterman global pairwise alignment in Python used to align reads to
    the 16S reference for per-position substitution counts
- name: PacBio
  role: CCS platform generating the full-length 16S reads whose substitution profile
    is extracted; its systematic deletion errors motivate the indel-exclusion step
- name: matplotlib
  role: Plots the per-position substitution frequency profiles reproducing Fig. 2a-b
    and the predicted seven-copy profile of Fig. 2c
- name: ruffus
  role: CGAT ruffus pipeline framework organizing the figure-level analysis code in
    the weinstock_full_length_16s repository
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:5812b8b5ed881b6b929828cfaab514cec13a328b154f98d3271f06ce1c8d3c0a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - matplotlib
  - minimap2
  - parasail
  - ruffus
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# indel-aware-alignment-postprocessing

## Summary

Post-process pairwise alignments of full-length 16S reads to a single 16S reference gene by tallying per-position substitution frequencies while deliberately excluding insertions and deletions, because systematic PacBio CCS deletion errors (notably in homopolymers) would otherwise be confounded with genuine intragenomic 16S copy polymorphisms.

## When to use

Use when aligning PacBio CCS full-length 16S or Illumina WGS reads to a single 16S reference gene and per-position substitution profiles of intragenomic 16S copies are needed, but raw indel tallies would be contaminated by platform-specific errors.

## When NOT to use

- Indel polymorphisms are the analysis target (e.g., validating a real copy-specific deletion) — this skill discards insertions and deletions by design
- Input reads are short amplicon reads of 16S variable regions (e.g., V4) rather than full-length ~1500 bp reads; intragenomic copy substitution profiles are not resolvable from sub-regions
- The goal is community-wide OTU clustering or taxonomic profiling rather than characterizing intragenomic 16S copy variation against a known reference genome

## Inputs

- PacBio CCS full-length 16S amplicon reads (FASTQ) from the mock community, from github.com/TheJacksonLaboratory/weinstock_full_length_16s
- Illumina WGS reads (FASTQ) of the same mock community, for independent stoichiometry validation
- Single E. coli K-12 MG1655 16S reference gene sequence (FASTA)
- RefSeq E. coli K-12 MG1655 genome from which the seven 16S gene copies are extracted

## Outputs

- substitution_profiles.csv: per-position substitution frequency table for PacBio-observed, Illumina-observed, and seven-copy-predicted profiles
- Figure 2a-c style plots of per-position substitution frequencies (observed PacBio, observed Illumina, predicted seven-copy stoichiometric) rendered with matplotlib

## How to apply

Align PacBio CCS full-length 16S reads — and separately Illumina WGS reads from github.com/TheJacksonLaboratory/weinstock_full_length_16s — to the single E. coli K-12 MG1655 16S reference gene with a global pairwise aligner (minimap2, or parasail/Smith-Waterman in Python). Tally per-position substitution frequencies from the alignments while deliberately discarding insertion and deletion events, since PacBio CCS produces systematic deletion errors coincident with homopolymer runs that otherwise masquerade as copy polymorphisms. Build the expected profile by extracting all seven 16S gene copies from the RefSeq E. coli K-12 MG1655 genome, aligning them to the same single reference, and converting copy stoichiometry into a predicted per-position substitution frequency profile. Write the three profiles (PacBio-observed, Illumina-observed, seven-copy-predicted) to substitution_profiles.csv and plot them with matplotlib to reproduce Fig. 2a-c. Treat agreement in substitution stoichiometry between the two platforms and the prediction as the pass criterion, acknowledging that the substitution-only filter cannot represent the one known deletion in one of the seven MG1655 16S genes.

## Related tools

- **minimap2** (Global aligner option named in the workflow for mapping PacBio CCS full-length 16S reads to the single E. coli K-12 MG1655 16S reference before substitution tallying)
- **parasail** (Smith-Waterman global pairwise alignment in Python used to align reads to the 16S reference for per-position substitution counts)
- **PacBio** (CCS platform generating the full-length 16S reads whose substitution profile is extracted; its systematic deletion errors motivate the indel-exclusion step)
- **matplotlib** (Plots the per-position substitution frequency profiles reproducing Fig. 2a-b and the predicted seven-copy profile of Fig. 2c)
- **ruffus** (CGAT ruffus pipeline framework organizing the figure-level analysis code in the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus

## Examples

```
minimap2 -ax map-pb ecoli_K12_MG1655_16S.fasta mock_pacbio_ccs_16s.fastq > pacbio_16s.sam
```

## Evaluation signals

- PacBio per-position substitution frequencies agree in stoichiometry with the independently derived Illumina WGS profile at the same 16S positions
- Both observed substitution profiles match the profile predicted from the seven RefSeq E. coli K-12 MG1655 16S copies (the Fig. 2a-c comparison)
- substitution_profiles.csv contains substitution-only tallies: no insertion or deletion contribution enters the per-position frequencies
- Substitution signal localizes to positions that differ among the seven extracted MG1655 16S copies rather than being distributed uniformly across the ~1500 bp gene
- Absence of any indel-derived signal is expected: the single known deletion in one of the seven MG1655 16S genes is, by design, not represented in the profile

## Limitations

- Ignoring insertions and deletions is an imperfect approach, demonstrated by the single deletion present in one of the seven E. coli strain K-12 substr. MG1655 16S genes
- Systematic PacBio CCS deletion errors coincident with homopolymer runs occur at a fixed frequency and may not improve with greater sequencing effort, limiting resolution of highly similar copies
- Full-length platforms are sufficiently accurate to resolve substitutions but not insertions/deletions between intragenomic 16S copies, so indel-based copy differences remain unresolvable

## Evidence

- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] Focusing on substitutions and ignoring insertions and deletions: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] Single deletion in one of the seven MG1655 16S genes: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [discussion] Systematic errors occur at a fixed frequency: "such systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort"
- [readme] Code organized as ruffus pipelines per figure: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
