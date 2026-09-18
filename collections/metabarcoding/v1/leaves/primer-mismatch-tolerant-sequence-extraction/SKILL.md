---
name: primer-mismatch-tolerant-sequence-extraction
description: Use when full-length 16S rRNA reference FASTA sequences (e.g., non-redundant
  Greengenes v13.8.99) must be trimmed into sub-region amplicons using common microbiome
  PCR primer pairs for in-silico taxonomic-accuracy comparison.
when_to_use_negative:
- Input is already sub-region amplicon sequence data or a feature/OTU table — trimming
  primers again is meaningless
- The goal is to analyze real sequencing runs (e.g., long-read Nanopore/PacBio mock
  or in-vivo communities) rather than an in-silico database comparison
- Reference sequences lack the primer-binding sites flanking the target variable region
  — no amplicon can be anchored and the extraction step cannot proceed
edam_operation: http://edamontology.org/operation_0294
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0091
tools:
- name: Cutadapt
  role: Performs primer-mismatch-tolerant (up to 3 mismatches) excision of sub-region
    amplicons from full-length 16S FASTA sequences, one run per primer set
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the study's analysis code,
    including the figure pipelines that wrapped this in-silico extraction
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:4793416dbb426d312a9cd367eabf18bbb42938cc2d0e7a0a0d60ff04ad6fbb68
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cutadapt
  - ruffus
  edam_operation: http://edamontology.org/operation_0294
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# primer-mismatch-tolerant-sequence-extraction

## Summary

Extracts in-silico 16S sub-region amplicons (V4, V1-V3, V3-V5, V6-V9, V1-V9) from full-length 16S reference sequences by trimming with primer-binding-site anchors, tolerating up to 3 primer mismatches. This recreates the exact sequence space a short-read amplicon run would sample, enabling taxonomic-resolution comparison against the full ~1500 bp gene.

## When to use

Use when full-length 16S rRNA reference FASTA sequences (e.g., non-redundant Greengenes v13.8.99) must be trimmed into sub-region amplicons using common microbiome PCR primer pairs for in-silico taxonomic-accuracy comparison.

## When NOT to use

- Input is already sub-region amplicon sequence data or a feature/OTU table — trimming primers again is meaningless
- The goal is to analyze real sequencing runs (e.g., long-read Nanopore/PacBio mock or in-vivo communities) rather than an in-silico database comparison
- Reference sequences lack the primer-binding sites flanking the target variable region — no amplicon can be anchored and the extraction step cannot proceed

## Inputs

- Non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA
- PCR primer pairs for the five sub-regions (V4, V1-V3, V3-V5, V6-V9, V1-V9) from Supplementary Table 1

## Outputs

- One filtered FASTA file per sub-region (V4, V1-V3, V3-V5, V6-V9, V1-V9) containing in-silico amplicons
- Per-sub-region amplicon length statistics (mean and standard deviation)

## How to apply

Load the non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA and retrieve the primer pairs for each of the five sub-regions from Supplementary Table 1. Run Cutadapt once per primer set, allowing a maximum of 3 mismatches to the primer-binding sites, so the amplicon spanning the targeted variable region is excised from every database sequence; sequences whose variable region cannot be anchored by either primer are discarded. Also discard any extracted sequence containing ambiguous 'N' bases. Compute the mean and standard deviation of amplicon lengths per sub-region and remove sequences longer or shorter than 2 standard deviations from the regional mean. Write the retained sequences to one filtered FASTA per sub-region (V4, V1-V3, V3-V5, V6-V9, V1-V9); the rationale is that the resulting files represent what short-read platforms targeting those variable regions would actually observe, against which full-length 16S taxonomic accuracy is compared.

## Related tools

- **Cutadapt** (Performs primer-mismatch-tolerant (up to 3 mismatches) excision of sub-region amplicons from full-length 16S FASTA sequences, one run per primer set)
- **ruffus** (CGAT ruffus pipeline framework used to organize the study's analysis code, including the figure pipelines that wrapped this in-silico extraction) — https://github.com/cgat-developers/ruffus

## Examples

```
cutadapt --no-indels -e 0.15 -g ^FWD_PRIMER...REV_PRIMER --discard-untrimmed greengenes_13_8_99_full_length.fasta > v4_amplicons.fasta
```

## Evaluation signals

- Exactly five filtered FASTA files are produced, one per sub-region (V4, V1-V3, V3-V5, V6-V9, V1-V9)
- Every retained sequence matched the sub-region primer set with no more than 3 mismatches (no amplicon-found-by-neither-primer sequences remain)
- No retained sequence contains ambiguous 'N' bases
- All retained amplicon lengths fall within 2 standard deviations of the per-sub-region mean, and the mean/SD stats were computed before filtering
- Amplicon length distributions are consistent with the known span of each targeted variable region

## Limitations

- Database sequences whose variable region is missing (no amplicon found by either primer) or that contain 'N' bases are dropped, so extracted sets are a filtered subset of Greengenes
- The ±2-SD length filter further reduces coverage of the database per sub-region
- Results reflect in-silico extraction only; the article notes none of the partial sub-regions recaptured the diversity of the full ~1500 bp gene, so amplicon sets should not be treated as equivalent to full-length data

## Evidence

- [other] Full-length 16S sequences containing primer-binding sites were trimmed to generate in-silico amplicons for different sub-regions: [evidence span withheld — anchor status: not_found]
- [intro] we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the: "we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [discussion] none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [readme] This repository contains code used to perform the analysis reported in Johnson et al.: "This repository contains code used to perform the analysis reported in Johnson et al."
