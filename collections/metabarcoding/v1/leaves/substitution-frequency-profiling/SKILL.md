---
name: substitution-frequency-profiling
description: Use when analyzing full-length 16S amplicon reads (PacBio CCS or Illumina
  WGS) to detect intragenomic 16S copy polymorphisms, or when validating observed
  per-position substitution frequencies against a reference genome's copy stoichiometry.
when_to_use_negative:
- When the goal is resolving intragenomic 16S copy differences mediated by insertions
  or deletions — the paper notes systematic PacBio CCS deletion errors at homopolymer
  runs make this unreliable, and one of the seven E. coli K-12 MG1655 16S genes carries
  a deletion that this substitution-only approach cannot capture.
- When input is short-read sequencing of a 16S variable region (e.g., V4), which cannot
  achieve the taxonomic resolution of the full ~1500 bp gene.
- When reads are from a species with a single intragenomic 16S copy, since no copy-stoichiometry
  prediction profile can be built.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0102
tools:
- name: PacBio
  role: Platform producing CCS full-length 16S reads whose per-position substitution
    frequencies are tallied for the substitution profile
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the figure-specific analysis
    pipelines in the source repository
  repo: https://github.com/cgat-developers/ruffus
- name: weinstock_full_length_16s
  role: Repository providing the mock community PacBio CCS and Illumina WGS read data
    and the analysis code underlying Figures 1-4
  repo: https://github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:76f5f2a7329bc5e6a8592d1df9d16c9a864dc63bf8f1cc78df9242c0fb87dbf7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - ruffus
  - weinstock_full_length_16s
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# substitution-frequency-profiling

## Summary

Builds per-position substitution frequency profiles of full-length 16S rRNA gene alignments (e.g., PacBio CCS reads aligned to a single 16S reference) while ignoring insertions/deletions, and validates the observed profile against the stoichiometric profile predicted from all intragenomic 16S copies in a reference genome (e.g., the seven copies in E. coli K-12 MG1655).

## When to use

Use when analyzing full-length 16S amplicon reads (PacBio CCS or Illumina WGS) to detect intragenomic 16S copy polymorphisms, or when validating observed per-position substitution frequencies against a reference genome's copy stoichiometry.

## When NOT to use

- When the goal is resolving intragenomic 16S copy differences mediated by insertions or deletions — the paper notes systematic PacBio CCS deletion errors at homopolymer runs make this unreliable, and one of the seven E. coli K-12 MG1655 16S genes carries a deletion that this substitution-only approach cannot capture.
- When input is short-read sequencing of a 16S variable region (e.g., V4), which cannot achieve the taxonomic resolution of the full ~1500 bp gene.
- When reads are from a species with a single intragenomic 16S copy, since no copy-stoichiometry prediction profile can be built.

## Inputs

- PacBio CCS full-length 16S FASTQ reads from a mock community (from github.com/TheJacksonLaboratory/weinstock_full_length_16s)
- Illumina WGS FASTQ reads of the same strain for stoichiometry validation
- Single 16S rRNA reference gene sequence (e.g., E. coli K-12 MG1655)
- RefSeq genome containing multiple 16S copies (e.g., E. coli K-12 MG1655 with seven 16S genes)

## Outputs

- Per-position substitution frequency table (substitution_profiles.csv) for PacBio and Illumina reads
- Matplotlib figure of per-position substitution frequency profiles reproducing Fig. 2a-c
- Predicted seven-copy stoichiometric substitution profile from aligned RefSeq 16S copies

## How to apply

Align full-length 16S reads (e.g., PacBio CCS reads of the mock community) to a single 16S reference gene sequence using a global pairwise aligner such as minimap2 or parasail/Smith-Waterman in Python. Tally per-position substitution frequencies from the alignments, deliberately restricting the analysis to substitutions and ignoring insertions and deletions to avoid platform-specific error contributions (notably PacBio CCS deletion errors coincident with homopolymer runs). Repeat the same tally for Illumina WGS reads against the same 16S reference to validate stoichiometry independently. Extract all intragenomic 16S copies from the RefSeq genome (e.g., seven copies for E. coli K-12 MG1655), align them to the single reference, and predict the expected per-position substitution frequency profile from the copy stoichiometry. Plot the PacBio and Illumina profiles alongside the predicted stoichiometric profile (reproducing Fig. 2a-c) with matplotlib and write the frequency table to substitution_profiles.csv; correctness is judged by agreement of both observed profiles with the predicted copy-stoichiometry profile.

## Related tools

- **PacBio** (Platform producing CCS full-length 16S reads whose per-position substitution frequencies are tallied for the substitution profile)
- **ruffus** (CGAT ruffus pipeline framework used to organize the figure-specific analysis pipelines in the source repository) — https://github.com/cgat-developers/ruffus
- **weinstock_full_length_16s** (Repository providing the mock community PacBio CCS and Illumina WGS read data and the analysis code underlying Figures 1-4) — https://github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Examples

```
minimap2 -ax asm5 ecoli_k12_16s_reference.fasta pacbio_ccs_16s.fastq > aln.sam; python tally_substitution_profile.py --bam aln.sam --out substitution_profiles.csv
```

## Evaluation signals

- The observed PacBio substitution profile mirrors exactly the profile predicted by aligning all seven RefSeq E. coli K-12 MG1655 16S copies to the single reference.
- The Illumina WGS substitution profile agrees in stoichiometry with both the PacBio profile and the predicted seven-copy profile.
- Insertion and deletion events are excluded from the tallies, so residual signal reflects substitutions rather than platform-specific errors.
- The frequency table substitution_profiles.csv is produced with per-position values consistent across the three profiles (PacBio, Illumina, predicted).

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences.
- Such systematic errors may occur at a given frequency and may not be improved by greater sequencing effort.
- Ignoring insertions/deletions is an imperfect approach, as demonstrated by a single deletion in one of the seven E. coli K-12 MG1655 16S genes.
- The study stopped short of determining how well the 99% identity threshold separates intragenomic vs. inter-genomic variation for all bacterial species.

## Evidence

- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] Phasing of 16S gene SNPs produced highly similar substitution profiles: "Phasing of 16S gene SNPs produced highly similar substitution profiles for closely related taxa"
- [discussion] The presence of a single deletion in one of the seven E. coli: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [readme] Code is organised into a set of ruffus pipelines: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
