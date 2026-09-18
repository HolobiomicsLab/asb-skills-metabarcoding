---
name: in-silico-pcr-amplicon-generation
description: Use when comparing taxonomic resolution of short-read 16S sub-region
  amplicons against full-length 16S, and you need sub-region FASTA files excised from
  full-length reference sequences using real PCR primer locations.
when_to_use_negative:
- Input sequences are already sub-region amplicons (e.g., V4 FASTQ reads) rather than
  full-length 16S reference sequences
- The analysis goal is long-read full-length 16S copy-variant phasing or 99% OTU clustering
  of full gene sequences, not sub-region excision
- You need wet-lab primer design or PCR optimization rather than in-silico trimming
  with existing published primer sets
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Cutadapt
  role: Excises sub-region amplicons from full-length 16S reference sequences by matching
    PCR primer-binding sites with a maximum of 3 mismatches
- name: ruffus
  role: CGAT ruffus framework used to organize the analysis pipelines underlying the
    paper's figures, including the in-silico amplicon workflow
  repo: github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:09822b391f7a7d1f01ad7d12395b9316e39bf9225d7d73ace33bd2249203372d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cutadapt
  - ruffus
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# in-silico-pcr-amplicon-generation

## Summary

Generates in-silico PCR amplicons for 16S sub-regions (V4, V1–V3, V3–V5, V6–V9, V1–V9) by trimming full-length Greengenes v13.8.99 reference sequences with Cutadapt primer sets, enabling taxonomic-resolution comparison between partial and full (~1500 bp) 16S sequencing.

## When to use

Use when comparing taxonomic resolution of short-read 16S sub-region amplicons against full-length 16S, and you need sub-region FASTA files excised from full-length reference sequences using real PCR primer locations.

## When NOT to use

- Input sequences are already sub-region amplicons (e.g., V4 FASTQ reads) rather than full-length 16S reference sequences
- The analysis goal is long-read full-length 16S copy-variant phasing or 99% OTU clustering of full gene sequences, not sub-region excision
- You need wet-lab primer design or PCR optimization rather than in-silico trimming with existing published primer sets

## Inputs

- Non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA (from the study repository)
- PCR primer pairs for each of the five sub-regions from Supplementary Table 1 (V4, V1–V3, V3–V5, V6–V9, V1–V9)

## Outputs

- One filtered amplicon FASTA file per sub-region: V4, V1-V3, V3-V5, V6-V9, and V1-V9
- Per-sub-region amplicon length mean and standard deviation statistics

## How to apply

Load the non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA from the study repository (TheJacksonLaboratory/weinstock_full_length_16s, run as CGAT ruffus pipelines) and retrieve the five primer pairs from Supplementary Table 1. Run Cutadapt separately for each sub-region primer set with a maximum of 3 mismatches to excise the amplicon spanning the targeted variable region from every database sequence. Discard sequences where no amplicon is found by either primer and sequences containing ambiguous 'N' bases. Compute the mean and standard deviation of amplicon lengths per sub-region and discard sequences longer or shorter than 2 standard deviations from the regional mean, then write one filtered FASTA per sub-region. The rationale is to approximate what short-read sequencing of each variable region would actually capture, so sub-region accuracy can be compared against the full ~1500 bp gene.

## Related tools

- **Cutadapt** (Excises sub-region amplicons from full-length 16S reference sequences by matching PCR primer-binding sites with a maximum of 3 mismatches)
- **ruffus** (CGAT ruffus framework used to organize the analysis pipelines underlying the paper's figures, including the in-silico amplicon workflow) — github.com/cgat-developers/ruffus

## Examples

```
cutadapt -e 0.2 --no-indels --discard-untrimmed -g ^V4_FWD_PRIMER -G ^V4_REV_PRIMER -o V4_amplicons.fasta gg_13_8_99_full_length_16s.fasta
```

## Evaluation signals

- Exactly five filtered FASTA files are produced, one per sub-region (V4, V1-V3, V3-V5, V6-V9, V1-V9)
- Every retained sequence contains a full primer match for the sub-region's primer pair with no more than 3 mismatches
- No retained sequence contains ambiguous 'N' bases
- All retained amplicon lengths fall within 2 standard deviations of that sub-region's mean amplicon length
- Per-sub-region mean and standard deviation of amplicon lengths are reported and consistent across runs

## Limitations

- Sequences whose variable region is missing (no amplicon found by either primer) are discarded, so poorly covered taxa are under-represented per sub-region
- Length filtering at ±2 standard deviations from the regional mean removes outlier amplicons that may carry genuine taxonomic signal
- The study showed none of the variable regions covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene, so sub-region results should not be treated as equivalent to full-length results

## Evidence

- [intro] In-silico dataset derived from Greengenes and primer trimming: "Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [readme] Pipelines organized with CGAT ruffus framework: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining
 to different figures. Each pipeline is designed to be run using a"
- [abstract] Short-read sub-regions cannot match full-length resolution: "We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene."
