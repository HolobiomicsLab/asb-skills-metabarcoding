---
name: long-read-to-reference-alignment
description: Use when full-length (~1500 bp) 16S PacBio CCS reads must be mapped to
  a single 16S reference to quantify per-position substitutions that distinguish intragenomic
  16S gene copies within one genome.
when_to_use_negative:
- The goal is to resolve intragenomic 16S differences involving insertions or deletions
  — PacBio CCS deletion errors coincident with homopolymer runs, and a single deletion
  in one of the seven E. coli K-12 MG1655 16S genes, make the substitutions-only approach
  imperfect
- The input reads are short reads covering only partial 16S variable regions (e.g.,
  V4), which the article shows cannot recapture the diversity of the full ~1500 bp
  gene
edam_operation: http://edamontology.org/operation_3182
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3174
tools:
- name: PacBio
  role: Platform producing the full-length 16S CCS reads that are aligned to the single
    16S reference to build substitution profiles
- name: minimap2
  role: Example aligner for globally aligning PacBio CCS full-length 16S reads to
    the single 16S reference gene sequence
- name: cgat-developers/ruffus
  role: Pipeline framework used to organise the analysis code into ruffus pipelines
    pertaining to different figures
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:42ad38a9dd199e8df061dcde77cc390e34e93acf6438b8f0864a08c726989b52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - cgat-developers/ruffus
  - minimap2
  edam_operation: http://edamontology.org/operation_3182
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# long-read-to-reference-alignment

## Summary

Aligns full-length PacBio CCS 16S rRNA reads to a single reference 16S gene sequence and tallies per-position substitution frequencies to resolve intragenomic 16S copy variation (e.g., the seven 16S copies of E. coli K-12 MG1655).

## When to use

Use when full-length (~1500 bp) 16S PacBio CCS reads must be mapped to a single 16S reference to quantify per-position substitutions that distinguish intragenomic 16S gene copies within one genome.

## When NOT to use

- The goal is to resolve intragenomic 16S differences involving insertions or deletions — PacBio CCS deletion errors coincident with homopolymer runs, and a single deletion in one of the seven E. coli K-12 MG1655 16S genes, make the substitutions-only approach imperfect
- The input reads are short reads covering only partial 16S variable regions (e.g., V4), which the article shows cannot recapture the diversity of the full ~1500 bp gene

## Inputs

- PacBio CCS full-length 16S rRNA gene reads (FASTQ) from a mock community
- Illumina WGS reads (FASTQ) from the same sample for stoichiometry validation
- Single 16S rRNA reference gene sequence (e.g., E. coli K-12 MG1655)
- RefSeq genome of the target organism to extract all 16S gene copies (e.g., seven copies in E. coli K-12 MG1655)

## Outputs

- Per-position substitution frequency profile for PacBio reads (Fig. 2a)
- Per-position substitution frequency profile for Illumina WGS reads (Fig. 2b)
- Predicted seven-copy stoichiometric substitution profile (Fig. 2c)
- Frequency table written to substitution_profiles.csv

## How to apply

Download mock community PacBio CCS full-length 16S reads and matching Illumina WGS reads from github.com/TheJacksonLaboratory/weinstock_full_length_16s, then align the PacBio reads to the single E. coli K-12 MG1655 16S reference gene using a global pairwise aligner (e.g., minimap2 or parasail/Smith-Waterman in Python). Tally per-position substitution frequencies from the alignments, deliberately restricting the analysis to substitutions and ignoring insertions and deletions to avoid platform-specific error contributions. Repeat the alignment and substitution tally for the Illumina WGS reads against the same 16S reference, and extract all seven 16S copies from the RefSeq genome, align them to the single reference, and predict the expected per-position substitution frequency profile from the copy stoichiometry. Judge correctness by verifying that the observed PacBio and Illumina substitution profiles agree in stoichiometry with the predicted seven-copy profile.

## Related tools

- **PacBio** (Platform producing the full-length 16S CCS reads that are aligned to the single 16S reference to build substitution profiles)
- **minimap2** (Example aligner for globally aligning PacBio CCS full-length 16S reads to the single 16S reference gene sequence)
- **cgat-developers/ruffus** (Pipeline framework used to organise the analysis code into ruffus pipelines pertaining to different figures) — https://github.com/cgat-developers/ruffus

## Examples

```
minimap2 -ax map-pb ecoli_k12_16s_ref.fasta pacbio_ccs_16s.fastq > aln.sam; python tally_substitutions.py --bam aln.bam --out substitution_profiles.csv --skip-indels
```

## Evaluation signals

- The observed PacBio substitution profile mirrors exactly the profile predicted by aligning all seven 16S sequences known to be present in the E. coli K-12 MG1655 reference genome
- The Illumina WGS per-position substitution profile validates the stoichiometry of the substitutions observed in the PacBio alignments
- The frequency table substitution_profiles.csv is produced and matches the plotted profiles of Fig. 2a-c
- Insertions and deletions are excluded from the tally, with per-position counts restricted to substitutions only

## Limitations

- PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences; such systematic errors may occur at a given frequency and may not be improved by greater sequencing effort
- Ignoring insertions/deletions is an imperfect approach, demonstrated by a single deletion in one of the seven E. coli K-12 MG1655 16S genes
- The approach is designed for substitutions only; full-length platforms are not accurate enough to resolve insertions/deletions between intragenomic 16S copies

## Evidence

- [discussion] focusing on substitutions and ignoring the contribution of insertions and deletions: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] PacBio CCS deletion errors limit resolution: "we highlighted the prevalence of sequencing errors in PacBio CCS reads as a factor that limits the ability to resolve highly similar sequences. A particular problem was deletion errors coincident"
- [discussion] single deletion in one of seven E. coli 16S copies: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [abstract] full-length platforms resolve substitutions not indels: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [readme] ruffus pipelines organise the repo code: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
