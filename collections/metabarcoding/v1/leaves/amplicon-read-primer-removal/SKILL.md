---
name: amplicon-read-primer-removal
description: Use when raw paired-end amplicon FASTQ files (e.g. 16S V4 with 515F/806R
  or ITS2 with F-ITS4/R-fITS7) still contain primer sequences at read ends and must
  be cleaned before DADA2 quality filtering and ASV inference.
when_to_use_negative:
- Input reads are already demultiplexed and primer-trimmed, so cutadapt would find
  no primer overlaps and only add overhead
- Input is not raw amplicon FASTQ (e.g. an already-denoised ASV table, contigs, or
  shotgun metagenomic reads)
- The locus uses degenerate primers not expressible within cutadapt's 2-primers-per-end
  and 0.2 mismatch-rate defaults as configured in dadasnake
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: dadasnake
  role: Snakemake workflow whose primer_cutting step drives cutadapt on the raw reads
  repo: https://github.com/a-h-b/dadasnake
- name: cutadapt
  role: performs the actual primer removal and poly-G tail removal from FASTQ reads
- name: conda
  role: environment manager used to install dadasnake and its snakemake-triggered
    tool environments
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/gigascience/giaa135
    title: Dadasnake, a Snakemake implementation of DADA2 to process amplicon sequencing
      data for microbial ecology
  source_content_hash: sha256:e9dc3e12ea7b7c5514657cc6dea83d30bce5b47ee55588ace11c447e9d0b94dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/a-h-b/dadasnake
  tools:
  - BLAST
  - conda
  - cutadapt
  - dadasnake
  - mothur
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# amplicon-read-primer-removal

## Summary

Removes primer sequences (and poly-G tails) from raw amplicon FASTQ reads with cutadapt as the first preprocessing step of the dadasnake workflow, so that downstream DADA2 denoising operates on primer-free amplicon sequences.

## When to use

Use when raw paired-end amplicon FASTQ files (e.g. 16S V4 with 515F/806R or ITS2 with F-ITS4/R-fITS7) still contain primer sequences at read ends and must be cleaned before DADA2 quality filtering and ASV inference.

## When NOT to use

- Input reads are already demultiplexed and primer-trimmed, so cutadapt would find no primer overlaps and only add overhead
- Input is not raw amplicon FASTQ (e.g. an already-denoised ASV table, contigs, or shotgun metagenomic reads)
- The locus uses degenerate primers not expressible within cutadapt's 2-primers-per-end and 0.2 mismatch-rate defaults as configured in dadasnake

## Inputs

- raw paired-end FASTQ files of amplicon libraries (e.g. ERR777696 16S or PRJNA626434 ITS2)
- dadasnake YAML configuration file specifying primer_cutting settings and paired=true
- tab-separated sample table with library, r1_file, r2_file columns
- primer sequences for the locus (e.g. 515F/806R or F-ITS4/R-fITS7)

## Outputs

- primer-trimmed (and poly-G-tail-removed) FASTQ reads feeding the DADA2 quality filtering step
- primer removal statistics reported by dadasnake's intermediary figures and tables

## How to apply

Configure the primer_cutting step in the dadasnake YAML config pointing to the raw FASTQ files of each library, with the sample table listing library, r1_file and r2_file columns. Run cutadapt with dadasnake's defaults: a minimum overlap of 10, a maximum of 2 primers per read end, and a mismatch rate of 0.2; the same step removes poly-G tails. Use primers appropriate to the locus (e.g. 515F GTGYCAGCMGCCGCGGTAA / 806R GGACTACNVGGGTWTCTAAT for 16S; F-ITS4 / R-fITS7 for fungal ITS2). Validate the configuration with a dadasnake dryrun (-d) before the real run so that mis-specified primers are caught before read processing.

## Related tools

- **dadasnake** (Snakemake workflow whose primer_cutting step drives cutadapt on the raw reads) — https://github.com/a-h-b/dadasnake
- **cutadapt** (performs the actual primer removal and poly-G tail removal from FASTQ reads)
- **conda** (environment manager used to install dadasnake and its snakemake-triggered tool environments)

## Evaluation signals

- A dadasnake dryrun (-d) completes without configuration or sample-table validation errors before the real run
- Downstream DADA2 error estimation and denoising proceed without primer-derived artefacts in inferred ASVs
- For the 59-species prokaryotic mock community (ERR777696), the full pipeline including primer removal yields 59 prokaryotic ASVs against SILVA v138
- For the 19-species fungal ITS2 mock community (PRJNA626434), the pipeline recovers exactly 19 fungal ASVs after primer removal with F-ITS4/R-fITS7
- dadasnake's intermediary statistics report read counts after primer removal consistent with expected primer occurrence rates

## Limitations

- Defaults allow a maximum of 2 primers per end and a 0.2 mismatch rate; loci with more primer variants per end or higher divergence need configuration changes
- Minimum overlap of 10 means partial primers shorter than 10 nt at read ends may not be removed
- Only trimming of primers and poly-G tails is covered here; contaminant or chimera handling happens in later dadasnake/DADA2 steps

## Evidence

- [methods] primer removal and removal of poly-G-tails - using cutadapt: "primer removal and removal of poly-G-tails - using cutadapt"
- [other] Remove the 515F (GTGYCAGCMGCCGCGGTAA) and 806R (GGACTACNVGGGTWTCTAAT) primers with cutadapt (minimum overlap 10, maximum 2 primers per end, 0.2 mismatch rate).: [evidence span withheld — anchor status: not_found]
- [other] Validate the configuration and sample table with a dadasnake dryrun (-d) before the real run.: [evidence span withheld — anchor status: not_found]
- [readme] dadasnake is a Snakemake workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables: "dadasnake is a [Snakemake](https://snakemake.readthedocs.io/en/stable/) workflow to process amplicon sequencing data, from raw fastq-files to taxonomically assigned "OTU" tables"
