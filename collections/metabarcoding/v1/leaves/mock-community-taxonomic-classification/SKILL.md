---
name: mock-community-taxonomic-classification
description: Use when you need to benchmark that a DADA2-based amplicon pipeline correctly
  recovers known community richness (e.g. exactly 59 prokaryotic ASVs) and genus composition
  from a mock community fastq dataset.
when_to_use_negative:
- Your input is already an ASV/feature table or has been taxonomically classified
  — this skill processes raw paired fastq reads from primer-trimming through classification.
- You lack a ground-truth mock community composition — without the 59-species reference
  there is no benchmark to compare recovered richness and genera against.
- Your data are single-end or non-16S fungal ITS amplicons requiring a different dadasnake
  configuration (e.g. UNITE instead of SILVA 138).
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: dadasnake
  role: Snakemake workflow executing the DADA2 pipeline (quality filtering, error
    estimation, denoising, merging, chimera removal) and taxonomic classification
    on the mock community data
  repo: https://github.com/a-h-b/dadasnake
- name: cutadapt
  role: Removes 515F/806R primers from the mock community reads (minimum overlap 10,
    0.2 mismatch rate)
- name: mothur
  role: Bayesian classifier (cutoff 60) assigning ASV taxonomy against SILVA_138 SSU
- name: BLAST
  role: Supports taxonomy assignment alongside mothur as per dadasnake default configuration
- name: DADA2
  role: Denoises and merges mock community read pairs and removes chimeras to delineate
    exact ASVs
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:d04cbf1656bcccac3399c4de7ed621ea58b6bd591378af21a4e45f9a3ab0f7fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - DADA2
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# mock-community-taxonomic-classification

## Summary

Validates an amplicon-processing pipeline (dadasnake/DADA2) by running the 16S rRNA mock community ERR777696 (59 prokaryotic species: 10 archaea, 49 bacteria) through primer removal, DADA2 denoising/merging, and mothur taxonomy assignment against SILVA 138, then comparing recovered ASVs to the known ground-truth composition.

## When to use

Use when you need to benchmark that a DADA2-based amplicon pipeline correctly recovers known community richness (e.g. exactly 59 prokaryotic ASVs) and genus composition from a mock community fastq dataset.

## When NOT to use

- Your input is already an ASV/feature table or has been taxonomically classified — this skill processes raw paired fastq reads from primer-trimming through classification.
- You lack a ground-truth mock community composition — without the 59-species reference there is no benchmark to compare recovered richness and genera against.
- Your data are single-end or non-16S fungal ITS amplicons requiring a different dadasnake configuration (e.g. UNITE instead of SILVA 138).

## Inputs

- Paired-end Illumina fastq files for the bacterial/archaeal mock community (ENA ERR777696)
- Tab-separated sample table with library, r1_file, r2_file columns
- dadasnake YAML configuration file (paired=true, raw_directory, SILVA v.138 database paths)
- SILVA_138 SSU reference database for taxonomy assignment

## Outputs

- ASV count table (tab-separated) from the dadasnake output directory
- Taxonomy assignment table (tab-separated, mothur Bayesian classification against SILVA_138)
- Validation result: inferred ASV count vs. 59-species ground truth and genus-level composition comparison

## How to apply

Prepare a dadasnake config (paired=true, raw_directory pointing to ERR777696 fastqs, a tab-separated sample table with library/r1_file/r2_file columns, and SILVA v.138 database paths). Remove the 515F (GTGYCAGCMGCCGCGGTAA) and 806R (GGACTACNVGGGTWTCTAAT) primers with cutadapt (minimum overlap 10, maximum 2 primers per end, 0.2 mismatch rate), then run the Snakemake-driven DADA2 steps: quality filtering/trimming, error estimation, denoising, merging (min_overlap 12, max_mismatch 0), and consensus-method chimera removal. Classify ASVs with the mothur Bayesian classifier at cutoff 60 against SILVA_138 SSU, with BLAST/BASTA support as per the default dadasnake configuration. Export the ASV count table and taxonomy files as tab-separated hand-off files from the output directory. Judge success by counting prokaryotic ASVs (59 expected, ignoring contaminants) and comparing genus-level taxa to the ground-truth mock composition, flagging the expected contaminants Corynebacterium, Staphylococcus, Rhizobiaceae, and Sphingomonas.

## Related tools

- **dadasnake** (Snakemake workflow executing the DADA2 pipeline (quality filtering, error estimation, denoising, merging, chimera removal) and taxonomic classification on the mock community data) — https://github.com/a-h-b/dadasnake
- **cutadapt** (Removes 515F/806R primers from the mock community reads (minimum overlap 10, 0.2 mismatch rate))
- **mothur** (Bayesian classifier (cutoff 60) assigning ASV taxonomy against SILVA_138 SSU)
- **BLAST** (Supports taxonomy assignment alongside mothur as per dadasnake default configuration)
- **DADA2** (Denoises and merges mock community read pairs and removes chimeras to delineate exact ASVs)

## Examples

```
cutadapt -g GTGYCAGCMGCCGCGGTAA -G GGACTACNVGGGTWTCTAAT -O 10 -e 0.2 -o trimmed_R1.fastq.gz -p trimmed_R2.fastq.gz ERR777696_R1.fastq.gz ERR777696_R2.fastq.gz
```

## Evaluation signals

- The pipeline infers exactly 59 prokaryotic ASVs (ignoring the expected contaminants) for the ERR777696 mock community.
- Genus-level composition of classified ASVs matches the ground-truth mock community composition after accounting for the four known contaminants (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas).
- Merge step reports merged read pairs consistent with min_overlap 12 and max_mismatch 0 settings, and chimera removal used the consensus method.
- Taxonomy table shows assignments from the mothur Bayesian classifier at cutoff 60 against SILVA_138 SSU, exported as tab-separated hand-off files.
- Processing intermediates (primer removal via cutadapt, quality filtering figures/tables) are reported by dadasnake without failed steps.

## Limitations

- Known contaminants (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas) inflate ASV counts above the 59-species ground truth and must be flagged rather than treated as misclassification.
- The mock community benchmark demonstrates dadasnake's default configurations for bacteria/archaea 16S; fungal or ITS datasets require adjusted configurations and databases (e.g. UNITE).
- Snakemake versions before 5.9.1 are untested; dadasnake is designed for cluster execution via conda and needs appropriate VARIABLE_CONFIG settings for submission systems.

## Evidence

- [methods] primer removal and removal of poly-G-tails - using cutadapt: "primer removal and removal of poly-G-tails - using cutadapt"
- [methods] error estimation and denoising via DADA2: "error estimation & denoising - using DADA2, including Novaseq-enabled models"
- [readme] dadasnake processes raw fastq to taxonomically assigned OTU tables: "dadasnake is a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables, based on the"
- [methods] SILVA database recommended for 16S data: "I'd suggest to use the [SILVA database](https://www.arb-silva.de/no_cache/download/archive/current/Exports/) for 16S data"
