---
name: amplicon-length-distribution-quality-control
description: Use when full-length 16S rRNA reference sequences (e.g., Greengenes v13.8.99
  FASTA) must be trimmed into per-sub-region in-silico amplicons and length-QC'd before
  benchmarking how well short-read variable regions match full-gene taxonomic resolution.
when_to_use_negative:
- Input is already an OTU/ASV feature table or amplicon read set rather than full-length
  16S reference sequences from a public database
- The goal is to analyze experimental long-read (PacBio/Nanopore) sequencing data
  rather than to build in-silico sub-region reference sets for taxonomic-accuracy
  comparison
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Cutadapt
  role: Trims each Greengenes full-length 16S sequence to the primer-bounded in-silico
    amplicon for each sub-region, allowing a maximum of 3 primer mismatches
- name: ruffus (CGAT framework)
  role: Pipeline framework used by the study repository (weinstock_full_length_16s)
    to organize the analysis code into runnable figure pipelines
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:b5814bc34fff08125dafecbf66b8b403139fe67c73d00f44d01fb021e984ae74
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cutadapt
  - ruffus (CGAT framework)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 3
---

# amplicon-length-distribution-quality-control

## Summary

Generates in-silico 16S sub-region amplicons (V4, V1–V3, V3–V5, V6–V9, V1–V9) from the non-redundant Greengenes v13.8.99 full-length 16S database by excising primer-bounded variable regions with Cutadapt, then quality-controls each set by 3-mismatch primer match, absence of ambiguous 'N' bases, and a per-region length filter (±2 standard deviations of the regional mean). This is the reference-preparation step underlying the paper's in-silico comparison of taxonomic accuracy between partial 16S sub-regions and the full ~1500 bp gene.

## When to use

Use when full-length 16S rRNA reference sequences (e.g., Greengenes v13.8.99 FASTA) must be trimmed into per-sub-region in-silico amplicons and length-QC'd before benchmarking how well short-read variable regions match full-gene taxonomic resolution.

## When NOT to use

- Input is already an OTU/ASV feature table or amplicon read set rather than full-length 16S reference sequences from a public database
- The goal is to analyze experimental long-read (PacBio/Nanopore) sequencing data rather than to build in-silico sub-region reference sets for taxonomic-accuracy comparison

## Inputs

- Non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA
- PCR primer pairs for the five sub-regions (V4, V1–V3, V3–V5, V6–V9, V1–V9) from Supplementary Table 1

## Outputs

- One filtered FASTA file of in-silico amplicons per sub-region: V4, V1–V3, V3–V5, V6–V9, and V1–V9
- Per-sub-region amplicon length mean and standard deviation statistics used for the ±2 SD filter

## How to apply

Load the non-redundant Greengenes v13.8.99 full-length 16S rRNA gene database FASTA from the study repository (TheJacksonLaboratory/weinstock_full_length_16s) and retrieve the PCR primer pairs for each of the five sub-regions from Supplementary Table 1. Run Cutadapt once per primer set, allowing a maximum of 3 mismatches, to excise the amplicon spanning the targeted variable region from every database sequence. Discard extracted sequences whose variable region is missing (no amplicon found by either primer) and any sequence containing ambiguous 'N' bases. For each sub-region, compute the mean and standard deviation of amplicon lengths and discard sequences longer or shorter than 2 standard deviations from the regional mean, writing the retained sequences to one filtered FASTA per sub-region (V4, V1–V3, V3–V5, V6–V9, V1–V9). The rationale is to emulate the exact amplicons a short-read study would obtain, so that downstream taxonomic-resolution comparisons against the full ~1500 bp gene are like-for-like; the paper found that none of these partial regions recaptured full-gene diversity.

## Related tools

- **Cutadapt** (Trims each Greengenes full-length 16S sequence to the primer-bounded in-silico amplicon for each sub-region, allowing a maximum of 3 primer mismatches)
- **ruffus (CGAT framework)** (Pipeline framework used by the study repository (weinstock_full_length_16s) to organize the analysis code into runnable figure pipelines) — https://github.com/cgat-developers/ruffus

## Examples

```
cutadapt -g ^PRIMER_FWD -a PRIMER_REV --no-indels --max-n 0 -o V4_amplicons.fasta greengenes_13_8_99_full_length.fasta
```

## Evaluation signals

- Exactly five filtered FASTA files are produced, one per sub-region: V4, V1–V3, V3–V5, V6–V9, and V1–V9
- Every retained sequence in each FASTA passes the 3-mismatch primer-match criterion for that region's primer pair
- No retained sequence contains ambiguous 'N' bases
- All retained sequences fall within 2 standard deviations of the sub-region's mean amplicon length
- Per-sub-region mean and standard deviation of amplicon lengths are computed and reported for the filter step

## Limitations

- Sub-regions whose sequences lack primer-binding sites (no amplicon found by either primer) are discarded entirely, shrinking the effective reference set per region
- The ±2 standard deviation length filter removes long/short outliers, so the retained amplicon sets are biased toward typical regional lengths
- The paper's own finding applies downstream: none of the variable regions covered by partial 16S sequencing recaptured the diversity of the full ~1500 bp gene, so these amplicons are not substitutes for full-length references

## Evidence

- [intro] Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4: "Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4"
- [discussion] none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [readme] Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
