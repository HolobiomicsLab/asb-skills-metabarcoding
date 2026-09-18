---
name: 16s-amplicon-taxonomy-assignment
description: Use when you have DADA2- or Deblur-denoised representative sequences
  (ASVs) from an amplicon run and need to label them taxonomically as part of a reproducible
  Tourmaline/QIIME 2 workflow before diversity analyses.
when_to_use_negative:
- Input is raw FASTQ rather than denoised representative sequences — the qaqc/dada2pe
  step must run first
- Single-end Deblur reads where paired-end DADA2 assumptions do not apply; Deblur
  is the single-end alternative in this workflow
- The goal is de novo OTU clustering rather than ASV-level taxonomic classification
edam_operation: http://edamontology.org/operation_0310
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: Tourmaline
  role: Snakemake-managed workflow that runs the taxonomy step via tourmaline.sh with
    its own Snakefile and config file
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the naive Bayes classify-sklearn plugin that assigns taxonomy to representative
    sequences
- name: Docker
  role: Container runtime for the Tourmaline workflow used to benchmark the taxonomy
    and other steps reproducibly
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:1facf961ba238226f0de0d5e26435dfe3036f89e1b7411cd6c26abb0483d2b39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Docker
  - QIIME 2
  - Tourmaline
  edam_operation: http://edamontology.org/operation_0310
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2
n_claims: 5
---

# 16s-amplicon-taxonomy-assignment

## Summary

Assigns taxonomy to denoised 16S rRNA representative sequences using QIIME 2's naive Bayes classify-sklearn, executed as the 'taxonomy' step of the Tourmaline Snakemake workflow against a curated reference database (e.g., SILVA 138.1).

## When to use

Use when you have DADA2- or Deblur-denoised representative sequences (ASVs) from an amplicon run and need to label them taxonomically as part of a reproducible Tourmaline/QIIME 2 workflow before diversity analyses.

## When NOT to use

- Input is raw FASTQ rather than denoised representative sequences — the qaqc/dada2pe step must run first
- Single-end Deblur reads where paired-end DADA2 assumptions do not apply; Deblur is the single-end alternative in this workflow
- The goal is de novo OTU clustering rather than ASV-level taxonomic classification

## Inputs

- Representative-sequence QIIME 2 artifact (ASVs) from the DADA2 (dada2pe) denoising step
- Reference database configured in the taxonomy config file (e.g., silva-138_1 or PR2)
- Sample metadata file and FASTQ manifest for PRJNA679730
- Tourmaline taxonomy step config file

## Outputs

- Taxonomic classification artifact for the representative sequences (QIIME 2 taxonomy annotation)
- Taxonomy visualization web-viewable via the HTML report and QIIME 2 viewer
- Per-step wall-clock timing recorded in runtimes.csv

## How to apply

Place the reference database (e.g., database_name: silva-138_1 or PR2), sample metadata, and manifest in the Tourmaline working directory along with the three step config files (qaqc, repseqs, taxonomy). Run the taxonomy step in isolation through tourmaline.sh/Snakemake with the taxonomy configfile and a chosen core count, which trains the QIIME 2 naive Bayes classifier (classify-sklearn) on the reference database and classifies the representative sequences produced by the denoising step. The step runs after qaqc and repseqs in the three-step (qaqc, repseqs, taxonomy) decomposition, and supports representative-sequence filtering by taxonomy if a taxonomic filter is desired. Use --cores N to parallelize; on the Lake Erie 16S rRNA dataset (PRJNA679730, ~96 samples, ~11.5 million paired reads), per-step wall-clock timings were recorded separately and the whole unfiltered workflow completed in ~5 hours with 1 core vs. ~2 hours with 8 cores. Verify the run by confirming the classified taxonomy artifact is produced, is viewable in the HTML report and QIIME 2 viewer, and that compiled per-step timings are written to runtimes.csv.

## Related tools

- **Tourmaline** (Snakemake-managed workflow that runs the taxonomy step via tourmaline.sh with its own Snakefile and config file) — github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the naive Bayes classify-sklearn plugin that assigns taxonomy to representative sequences)
- **Docker** (Container runtime for the Tourmaline workflow used to benchmark the taxonomy and other steps reproducibly)

## Examples

```
./tourmaline.sh --step taxonomy --configfile [config1,config2,config3] --cores 8
```

## Evaluation signals

- The taxonomy step completes without error when invoked with --step taxonomy and the correct configfile, using its dedicated Snakefile
- The classified taxonomy artifact renders as a web-viewable visualization linked in the generated HTML report
- Assignments are consistent with the specified reference database (e.g., silva-138_1) in the taxonomy config file
- Per-step wall-clock time is recorded; total unfiltered runtime on the Lake Erie PRJNA679730 dataset approximates ~5 h with --cores 1 and ~2 h with --cores 8
- Timings are compiled into runtimes.csv with one row per main workflow step

## Limitations

- The taxonomy step must be run in the tourmaline directory and requires its own Snakefile and config file; using the wrong Snakefile for an individual rule fails
- Reference database choice (silva-138_1 vs. PR2) changes classifications and must match the amplicon target; no changelog is available for version drift ('No changelog found')
- Runtime figures are machine- and dataset-specific (~11.5 million paired reads, Lake Erie 2018); scaling to other datasets will differ with core count and hardware

## Evidence

- [abstract] Tourmaline assigns taxonomy to denoised representative sequences as a distinct workflow step: "assigns taxonomy to the resulting representative sequences"
- [abstract] Features include support for multiple cores, automatic determination of trimming parameters using quality scores, representative sequence filtering (taxonomy, length, abundance, prevalence, or ID): "Features include support for multiple cores, automatic determination of trimming parameters using quality scores, representative sequence filtering (taxonomy, length, abundance, prevalence, or ID)"
- [methods] SILVA 138.1 is the reference database configured for taxonomy assignment: "database_name: silva-138_1"
- [readme] tourmaline.sh is the entry point for running the taxonomy step with configurable cores: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [readme] Each of the three steps has its own Snakefile, so the correct one must be specified when running an individual rule: "You can still run individual snakemake rules as before. Each of the three steps (explained more below) has its own Snakefile, so you must specify the correct snakefile"
