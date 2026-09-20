---
name: itsx-fungal-amplicon-filtering
description: Use when processing paired-end fungal amplicon FASTQ data (e.g. ITS1
  primers, as in BioProject PRJNA272747) with dadasnake and you need reads restricted
  to the ITS1 subregion before taxonomic assignment.
when_to_use_negative:
- Input is 16S bacterial/archaeal amplicon data — use a SILVA-based classification
  setup instead of ITSx fungal ITS1 filtering
- Data do not target the ITS region (ITSx subregion extraction is specific to ITS
  amplicons)
- Input is already a filtered ASV/OTU feature table rather than raw FASTQ reads —
  dadasnake processes from raw fastq-files
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: dadasnake
  role: 1-command Snakemake pipeline wrapping the full ITS1 workflow, including ITSx
    filtering and mothur classification
  repo: https://github.com/a-h-b/dadasnake
- name: Snakemake
  role: Workflow engine that parallelizes the ITS1 processing steps across cores (e.g.
    4 vs 15 threads)
- name: DADA2
  role: Quality filtering/trimming, error estimation and denoising, and chimera removal
    before ITSx subregion extraction
- name: cutadapt
  role: Primer removal and removal of poly-G tails from raw paired-end reads
- name: mothur
  role: Bayesian taxonomic classification of ITS1 ASVs against a UNITE-formatted fungal
    ITS database
- name: conda
  role: Environment management for installing and running dadasnake
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:5a3d3512c2d8f66b765866fe8c58183f238702c4f8c234805b4dcc138ed3d3bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - DADA2
  - ITSx
  - Snakemake
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# itsx-fungal-amplicon-filtering

## Summary

Filter amplicon reads to the fungal ITS1 region within the dadasnake pipeline, using ITSx region extraction combined with DADA2-based ASV inference and UNITE/mothur-based fungal taxonomic classification.

## When to use

Use when processing paired-end fungal amplicon FASTQ data (e.g. ITS1 primers, as in BioProject PRJNA272747) with dadasnake and you need reads restricted to the ITS1 subregion before taxonomic assignment.

## When NOT to use

- Input is 16S bacterial/archaeal amplicon data — use a SILVA-based classification setup instead of ITSx fungal ITS1 filtering
- Data do not target the ITS region (ITSx subregion extraction is specific to ITS amplicons)
- Input is already a filtered ASV/OTU feature table rather than raw FASTQ reads — dadasnake processes from raw fastq-files

## Inputs

- Raw paired-end FASTQ files of fungal ITS1 amplicons (e.g. 267 samples / 46.8 million reads from NCBI SRA BioProject PRJNA272747)
- Tab-separated sample table with 'library', 'r1_file', and 'r2_file' column headers
- dadasnake config.yaml with paired=true, ITSx enabled and region=ITS1, and mothur classification against a UNITE-formatted fungal ITS database
- UNITE reference database for ITS classification

## Outputs

- ITS1-restricted, chimera-removed ASV/OTU table (taxonomically assigned via mothur against UNITE)
- Standard-format hand-off files: biom format, phyloseq objects, text files, or R data sets
- Intermediary statistics reported in figures and tables
- 'workflow.done' marker file confirming pipeline completion

## How to apply

Build a tab-separated sample table with 'library', 'r1_file', and 'r2_file' headers pointing to the raw paired-end FASTQ files, then write a dadasnake config.yaml in which paired is set to true, ITSx is enabled with region set to ITS1, and mothur-based Bayesian classification is activated against a fungal ITS (UNITE-formatted) database. Run the standard dadasnake chain — cutadapt primer removal (and poly-G tail removal), DADA2 quality filtering and trimming, error estimation and denoising, pair merging (min_overlap at least 20 bp), chimera removal, then ITSx filtering to retain only ITS1 sequences — followed by mothur classification. Validate the config and inputs with a dryrun (dadasnake -d) before committing compute, and confirm completion via the 'workflow.done' marker file in the output directory. The final taxonomically assigned OTU/ASV tables are handed off in standard formats (biom, phyloseq, text or R data sets).

## Related tools

- **dadasnake** (1-command Snakemake pipeline wrapping the full ITS1 workflow, including ITSx filtering and mothur classification) — https://github.com/a-h-b/dadasnake
- **Snakemake** (Workflow engine that parallelizes the ITS1 processing steps across cores (e.g. 4 vs 15 threads))
- **DADA2** (Quality filtering/trimming, error estimation and denoising, and chimera removal before ITSx subregion extraction)
- **cutadapt** (Primer removal and removal of poly-G tails from raw paired-end reads)
- **mothur** (Bayesian taxonomic classification of ITS1 ASVs against a UNITE-formatted fungal ITS database)
- **conda** (Environment management for installing and running dadasnake)

## Examples

```
dadasnake -d  # dryrun to confirm sample table, ITSx region=ITS1 config, and inputs before the full run
```

## Evaluation signals

- Presence of the 'workflow.done' marker file in the output directory after each run
- Retained ASVs fall within expected ITS1 length ranges after ITSx subregion extraction (length check performed in R)
- Chimera removal and read-pair merging succeed with min_overlap of at least 20 bp
- Taxonomic assignments are produced against the UNITE-formatted fungal ITS database via mothur
- Reported runtime matches the benchmark scale: just under 4 hours on four 8 GB cores for 267 samples / 46.8M reads, with ~2x speedup at 15 cores

## Limitations

- dadasnake's taxonomic classification setups have been tested with specific reference versions (e.g. fungalTraits v1.2, UNITE); other database versions may need re-validation
- Runtime scales with sample count and read depth; parallelization gains depend on available cores (measured ~2x from 4 to 15 cores)
- Dadasnake requires Snakemake version >= 5.9.1 and has not been tested with previous versions

## Evidence

- [methods] UNITE for ITS: "and [UNITE](https://doi.org//10.15156/BIO/786336) for ITS"
- [methods] chimera removal via DADA2: "chimera removal - using DADA2"
- [methods] primer removal with cutadapt: "primer removal and removal of poly-G-tails - using cutadapt"
- [readme] raw fastq to OTU tables: "workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables, based on the [DADA2](http://benjjneb.github.io/dada2/) method."
- [intro] output formats: "Final data output formats include biom format, phyloseq objects, and flexible text files or R data sets for easy integration in microbial ecology analysis scripts"
