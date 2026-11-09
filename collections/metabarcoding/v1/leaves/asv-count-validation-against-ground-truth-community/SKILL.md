---
name: asv-count-validation-against-ground-truth-community
description: Use when a mock community amplicon library with known composition (e.g.
  10 archaea + 49 bacteria, ENA ERR777696) has been denoised with dadasnake/DADA2
  and you must quantify richness and composition recovery accuracy.
when_to_use_negative:
- Input is an environmental or unknown-composition sample with no ground-truth species
  list — there is no reference richness to validate against
- Input is already an ASV/OTU count table (final_table_filtering stage); the skill
  assumes raw FASTQ through dadasnake, not post hoc tables
- Data are not prokaryotic amplicons (e.g. ITS/fungal mock communities), since the
  59-ASV expectation and contaminant panel are specific to the bacterial/archaeal
  mock
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: dadasnake
  role: Runs the DADA2 workflow that produces the ASV count and taxonomy tables being
    validated
  repo: https://github.com/a-h-b/dadasnake
- name: DADA2
  role: Denoising, merging (min_overlap 12, max_mismatch 0) and consensus chimera
    removal that generate the ASVs
- name: cutadapt
  role: Removes 515F/806R primers (minimum overlap 10, max 2 primers per end, 0.2
    mismatch rate) so ASV richness is not inflated
- name: mothur
  role: Bayesian classifier (cutoff 60) against SILVA_138 SSU supplying genus labels
    for composition comparison
- name: Snakemake
  role: Orchestrates the reproducible workflow whose output ASV table is validated
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:7f2fad8f46439d398d3ebe3e8e779ce1e2a92743012072f1a5b4a3dad636ac21
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - DADA2
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# asv-count-validation-against-ground-truth-community

## Summary

Validates DADA2/dadasnake ASV inference against a ground-truth mock community by counting inferred prokaryotic ASVs and comparing genus-level composition to the expected 59-species bacterial/archaeal mixture, flagging known contaminants.

## When to use

Use when a mock community amplicon library with known composition (e.g. 10 archaea + 49 bacteria, ENA ERR777696) has been denoised with dadasnake/DADA2 and you must quantify richness and composition recovery accuracy.

## When NOT to use

- Input is an environmental or unknown-composition sample with no ground-truth species list — there is no reference richness to validate against
- Input is already an ASV/OTU count table (final_table_filtering stage); the skill assumes raw FASTQ through dadasnake, not post hoc tables
- Data are not prokaryotic amplicons (e.g. ITS/fungal mock communities), since the 59-ASV expectation and contaminant panel are specific to the bacterial/archaeal mock

## Inputs

- Paired-end Illumina FASTQ files of the mock community library (e.g. ERR777696)
- dadasnake config YAML with paired=true, raw_directory and sample table (library/r1_file/r2_file)
- SILVA v.138 SSU classification database (e.g. silva_nr99_v138_train_set.fa.gz)
- ASV count table and taxonomy table exported in tab-separated format from the dadasnake output directory
- Ground-truth mock community composition table (available from Zenodo [60])

## Outputs

- Total prokaryotic ASV count compared to the 59-species ground truth
- Genus-level taxa list matched against the expected mock composition
- List of flagged contaminant genera (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas)

## How to apply

After running dadasnake end-to-end (primer removal of 515F GTGYCAGCMGCCGCGGTAA and 806R GGACTACNVGGGTWTCTAAT with cutadapt at minimum overlap 10 and 0.2 mismatch rate, DADA2 quality filtering, error estimation, denoising, merging with min_overlap 12 and max_mismatch 0, consensus chimera removal), export the ASV count table and taxonomy as tab-separated files. Count the inferred ASVs and compare against the expected ground truth of 59 prokaryotic ASVs, ignoring contaminant taxa. List genus-level taxa from the mothur Bayesian classification (cutoff 60) against SILVA_138 SSU, then subtract expected contaminants (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas) before judging composition accuracy. A correct run yields exactly 59 prokaryotic ASVs and a genus composition matching the mock after accounting for the four contaminants.

## Related tools

- **dadasnake** (Runs the DADA2 workflow that produces the ASV count and taxonomy tables being validated) — https://github.com/a-h-b/dadasnake
- **DADA2** (Denoising, merging (min_overlap 12, max_mismatch 0) and consensus chimera removal that generate the ASVs)
- **cutadapt** (Removes 515F/806R primers (minimum overlap 10, max 2 primers per end, 0.2 mismatch rate) so ASV richness is not inflated)
- **mothur** (Bayesian classifier (cutoff 60) against SILVA_138 SSU supplying genus labels for composition comparison)
- **Snakemake** (Orchestrates the reproducible workflow whose output ASV table is validated)

## Examples

```
git clone https://github.com/a-h-b/dadasnake.git; cd dadasnake; ./dadasnake  # then export ASV count/taxonomy TSVs and compare to the 59-species ground truth
```

## Evaluation signals

- The inferred ASV count equals 59 prokaryotic ASVs (contaminants ignored) for ERR777696
- Genus-level composition after removing Corynebacterium, Staphylococcus, Rhizobiaceae and Sphingomonas matches the ground-truth mock composition
- ASV table and taxonomy export successfully as tab-separated files from the dadasnake output directory
- Chimera removal (consensus method) and merge parameters (min_overlap 12, max_mismatch 0) were applied without leaving spurious ASVs
- Each expected mock genus (10 archaea, 49 bacteria) is represented by at least one ASV in the count table

## Limitations

- Known contaminants (Corynebacterium, Staphylococcus, Rhizobiaceae, Sphingomonas) inflate richness and must be excluded before counting toward the 59-ASV expectation
- Validation is library-specific (ERR777696, the largest Illumina library of the 59-species mock); other libraries/platforms need their own ground truth
- Only the largest library was tested against exactly 59 ASVs; smaller mock libraries may not reach the full species count

## Evidence

- [abstract] Suitability of default configurations for bacteria/archaea mock: "The suitability of the provided default configurations is demonstrated using mock community data from bacteria and archaea, as well as fungi."
- [abstract] dadasnake single-command pipeline wrapping DADA2: "We present dadasnake, a user-friendly, 1-command Snakemake pipeline that wraps the preprocessing of sequencing reads and the delineation of exact sequence variants"
- [readme] Output formats for hand-off and validation: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts."
- [readme] Intermediary statistics reporting: "dadasnake reports on intermediary steps and statistics in intuitive figures and tables."
- [methods] Chimera removal step: "chimera removal - using DADA2"
