---
name: reference-database-fasta-processing
description: Use when a taxonomic-resolution or primer-bias comparison requires excising
  sub-region 16S amplicons (e.g., V4, V3-V5) in silico from a full-length reference
  FASTA using known primer-binding sites.
when_to_use_negative:
- Input sequences are already sub-region amplicon reads (e.g., V4 FASTQ from a sequencing
  run) rather than a full-length reference FASTA — trimming primers again is unnecessary
- The goal is strain- or species-level analysis of real sequencing data with intragenomic
  16S copy variants, not an in-silico database comparison
- The reference database lacks primer-binding sites or contains sequences outside
  the targeted region, since no valid amplicon can be excised and everything would
  be discarded
edam_operation: http://edamontology.org/operation_0231
edam_topics: []
tools:
- name: Cutadapt
  role: Excises the in-silico amplicon for each primer set (max 3 mismatches) from
    every Greengenes full-length 16S sequence
- name: ruffus
  role: CGAT ruffus pipeline framework used to organise the study's analysis pipelines
    into staged tasks, including these database-trimming steps
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:8d6a3d7fb17a5f0287431c7f02366cadbd72eef15dc8bfaf807a064b0942b05c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - CUTADAPT
  - Cutadapt
  - Mesquite
  - ruffus
  edam_operation: http://edamontology.org/operation_0231
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# reference-database-fasta-processing

## Summary

Generate in-silico sub-region amplicons (V4, V1-V3, V3-V5, V6-V9, V1-V9) from the non-redundant Greengenes v13.8.99 full-length 16S rRNA gene FASTA by trimming each sequence with Cutadapt at commonly used PCR primer pairs, enabling fair comparison of taxonomic resolution between partial and full-length 16S sequencing.

## When to use

Use when a taxonomic-resolution or primer-bias comparison requires excising sub-region 16S amplicons (e.g., V4, V3-V5) in silico from a full-length reference FASTA using known primer-binding sites.

## When NOT to use

- Input sequences are already sub-region amplicon reads (e.g., V4 FASTQ from a sequencing run) rather than a full-length reference FASTA — trimming primers again is unnecessary
- The goal is strain- or species-level analysis of real sequencing data with intragenomic 16S copy variants, not an in-silico database comparison
- The reference database lacks primer-binding sites or contains sequences outside the targeted region, since no valid amplicon can be excised and everything would be discarded

## Inputs

- Non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA (from the study repository)
- Primer pairs for five 16S sub-regions (V4, V1-V3, V3-V5, V6-V9, V1-V9) from Supplementary Table 1

## Outputs

- One filtered in-silico amplicon FASTA file per sub-region: V4, V1-V3, V3-V5, V6-V9, V1-V9
- Per-sub-region mean and standard deviation of amplicon lengths used for the 2-SD length filter

## How to apply

Load the non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA and retrieve the five primer pairs for the sub-regions from Supplementary Table 1. Run Cutadapt once per primer set, allowing a maximum of 3 mismatches to the primer-binding sites, to excise the amplicon spanning each targeted variable region from every database sequence. Discard extracted sequences whose variable region is missing (no amplicon found by either primer) and any sequences containing ambiguous 'N' bases. For each sub-region, compute the mean and standard deviation of amplicon lengths and discard sequences outside 2 standard deviations of the regional mean, then write one filtered FASTA file per sub-region (V4, V1-V3, V3-V5, V6-V9, V1-V9). Rationale: comparing like-for-like amplicons against the ~1500 bp full gene isolates the effect of sub-region choice on taxonomic accuracy rather than sequence-source differences.

## Related tools

- **Cutadapt** (Excises the in-silico amplicon for each primer set (max 3 mismatches) from every Greengenes full-length 16S sequence)
- **ruffus** (CGAT ruffus pipeline framework used to organise the study's analysis pipelines into staged tasks, including these database-trimming steps) — https://github.com/cgat-developers/ruffus

## Examples

```
cutadapt -g ^FWD_PRIMER -G ^REV_PRIMER --no-indels -e 0.05 --discard-untrimmed -o V4_amplicons.fasta greengenes_13_8_99_16S.fasta
```

## Evaluation signals

- Exactly five filtered FASTA files are produced, one per sub-region (V4, V1-V3, V3-V5, V6-V9, V1-V9)
- Every retained sequence contains a primer match within 3 mismatches for both primers of its sub-region
- No retained sequence contains ambiguous 'N' bases
- All retained amplicon lengths fall within 2 standard deviations of the sub-region mean amplicon length
- Sequences with a missing variable region (no amplicon found by either primer) are absent from the output

## Limitations

- Sequences lacking primer-binding sites or containing the sub-region variable region are discarded, shrinking the effective reference set per sub-region
- In-silico amplicons model only sequences present in the database and primer specificity within 3 mismatches; they cannot recapitulate real amplicon sequencing error profiles
- None of the variable regions covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene, so outputs are inherently resolution-limited compared to full-length sequences

## Evidence

- [methods] Run Cutadapt for each primer set with a maximum of 3 mismatches to excise the amplicon spanning the targeted variable region from every database sequence.: [evidence span withheld — anchor status: not_found]
- [methods] 2-SD length filter: [evidence span withheld — anchor status: not_found]
- [readme] ruffus pipeline organisation: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
