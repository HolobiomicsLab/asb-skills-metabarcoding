---
name: multi-copy-gene-stoichiometry-comparison
description: Use when full-length 16S reads (e.g., PacBio CCS ~1500 bp) are available
  for an organism whose reference genome contains multiple intragenomic 16S copies,
  and you need to check that observed per-position substitution frequencies match
  copy-stoichiometry predictions.
when_to_use_negative:
- The question depends on insertions or deletions between 16S copies — indels are
  deliberately excluded here because PacBio CCS deletion errors coincide with homopolymer
  runs
- Only short-read data covering a single 16S variable region (e.g., V4) are available;
  such regions cannot recapture full-gene diversity or intragenomic copy profiles
- The organism's genome contains a single 16S copy, so no intragenomic stoichiometric
  prediction can be built
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: PacBio
  role: Sequencing platform generating full-length (~1500 bp) 16S CCS reads whose
    substitution profile is compared against the seven-copy prediction
- name: minimap2
  role: Global pairwise aligner used to align PacBio CCS and Illumina WGS reads to
    the single E. coli K-12 MG1655 16S reference gene for per-position substitution
    tallying
- name: ruffus
  role: Python pipeline framework used to organise the analysis code into ruffus pipelines
    pertaining to different figures
  repo: https://github.com/cgat-developers/ruffus
- name: weinstock_full_length_16s
  role: Repository supplying the mock community PacBio CCS 16S reads and Illumina
    WGS reads, plus the per-figure analysis code
  repo: https://github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:c2fa6bfe4a9c1f59c824db2d071566ffd86f1f730752fe548f0d36ea2e8e6c9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - minimap2
  - ruffus
  - weinstock_full_length_16s
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# multi-copy-gene-stoichiometry-comparison

## Summary

Validates that long-read (PacBio CCS) or short-read sequencing of a multi-copy gene reproduces the substitution profile predicted from the stoichiometry of all gene copies in a reference genome, using E. coli K-12 MG1655's seven 16S copies as the benchmark. Restricting analysis to substitutions (ignoring indels) avoids platform-specific PacBio CCS deletion errors.

## When to use

Use when full-length 16S reads (e.g., PacBio CCS ~1500 bp) are available for an organism whose reference genome contains multiple intragenomic 16S copies, and you need to check that observed per-position substitution frequencies match copy-stoichiometry predictions.

## When NOT to use

- The question depends on insertions or deletions between 16S copies — indels are deliberately excluded here because PacBio CCS deletion errors coincide with homopolymer runs
- Only short-read data covering a single 16S variable region (e.g., V4) are available; such regions cannot recapture full-gene diversity or intragenomic copy profiles
- The organism's genome contains a single 16S copy, so no intragenomic stoichiometric prediction can be built

## Inputs

- PacBio CCS full-length 16S sequencing reads (FASTQ/FASTA) from a mock community
- Illumina WGS reads for the same organism (validation of substitution stoichiometry)
- RefSeq genome of the target organism (e.g., E. coli K-12 MG1655) to extract the seven 16S gene copies
- Single reference 16S gene sequence (FASTA) used as alignment target

## Outputs

- Per-position substitution frequency table (substitution_profiles.csv) for PacBio, Illumina, and the predicted seven-copy stoichiometric profile
- Per-position substitution frequency plots reproducing Fig. 2a-b (observed) and Fig. 2c (predicted)
- Validation verdict that observed PacBio and Illumina profiles match the predicted intragenomic copy stoichiometry

## How to apply

Extract all 16S gene copies from the RefSeq genome (e.g., the seven copies in E. coli K-12 MG1655), align each to a single 16S reference sequence, and compute the expected per-position substitution frequency from the copy stoichiometry. Independently, align platform reads (PacBio CCS full-length 16S, plus Illumina WGS for validation) to the same single 16S reference with a global pairwise aligner such as minimap2 or parasail/Smith-Waterman in Python, and tally per-position substitution frequencies. Deliberately ignore insertions and deletions when tallying, since PacBio CCS deletion errors coincide with homopolymer runs and would confound copy-variant signals. Plot the PacBio and Illumina per-position substitution profiles (Fig. 2a-b) alongside the predicted seven-copy stoichiometric profile (Fig. 2c), and write frequencies to substitution_profiles.csv. The skill is applied correctly when the observed profiles agree in stoichiometry with the copy-derived prediction, and Illumina WGS independently corroborates the substitution frequencies.

## Related tools

- **PacBio** (Sequencing platform generating full-length (~1500 bp) 16S CCS reads whose substitution profile is compared against the seven-copy prediction)
- **minimap2** (Global pairwise aligner used to align PacBio CCS and Illumina WGS reads to the single E. coli K-12 MG1655 16S reference gene for per-position substitution tallying)
- **ruffus** (Python pipeline framework used to organise the analysis code into ruffus pipelines pertaining to different figures) — https://github.com/cgat-developers/ruffus
- **weinstock_full_length_16s** (Repository supplying the mock community PacBio CCS 16S reads and Illumina WGS reads, plus the per-figure analysis code) — https://github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Examples

```
minimap2 -a ecoli_K12_16S_ref.fa pacbio_ccs_16s.fastq > pacbio_vs_16S.sam  # then tally per-position substitution frequencies, excluding indels
```

## Evaluation signals

- The PacBio-observed per-position substitution profile mirrors exactly the profile predicted by aligning all seven 16S sequences from the E. coli K-12 MG1655 RefSeq genome to the single 16S reference
- Illumina WGS reads yield substitution frequencies at the same positions with the same stoichiometry as the PacBio profile
- Substitution tallies exclude insertions and deletions; indels are not counted toward copy-variant signals
- A substitution_profiles.csv is produced containing per-position frequencies for observed (PacBio, Illumina) and predicted (seven-copy stoichiometric) profiles

## Limitations

- PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences; systematic errors may occur at a given frequency and may not be improved by greater sequencing effort
- Ignoring insertions/deletions is an imperfect approach, demonstrated by a single deletion in one of the seven E. coli K-12 MG1655 16S genes
- This benchmark covers substitutions only; it does not establish how well a 99% identity threshold separates intragenomic from inter-genomic variation across all bacterial species

## Evidence

- [abstract] Full-length sequencing platforms are accurate enough to resolve subtle nucleotide substitutions, but not insertions/deletions, between intragenomic 16S copies: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] Restricting analysis to substitutions avoids platform-specific PacBio deletion errors: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] Ignoring indels is imperfect: one of the seven E. coli 16S genes contains a deletion: "The presence of a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes demonstrates that this is an imperfect approach."
- [readme] Analysis code is organised as ruffus pipelines per figure: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
