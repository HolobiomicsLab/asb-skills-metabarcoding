---
name: taxonomic-classification-assignment
description: Use when a Tourmaline run has produced a feature table and representative-sequence
  artifacts from DADA2/Deblur denoising and taxonomy must be assigned to the ASVs
  as the next workflow step before diversity analyses.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been denoised — run the DADA2/Deblur
  denoising step (qaqc/repseqs) first
- The workflow run is Tourmaline 2, where taxonomy should be invoked via the tourmaline.sh
  script with --step taxonomy rather than standalone V1 pseudo-rules
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: QIIME 2
  role: Runs the taxonomy classifiers that assign taxonomy to the denoised representative
    sequences
  repo: https://github.com/aomlomics/tourmaline
- name: Tourmaline
  role: Snakemake-based workflow wrapping the QIIME 2 taxonomy step and generating
    the HTML report linking the taxonomy outputs
  repo: https://github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system executing the taxonomy rule (e.g. dada2_pe_taxonomy_unfiltered)
  repo: https://github.com/aomlomics/tourmaline
- name: DADA2
  role: Upstream denoising step producing the representative sequences that taxonomy
    is assigned to
  repo: https://github.com/benjjneb/dada2
- name: QIIME 2 viewer
  role: Web-viewable inspection of taxonomy visualizations produced by the workflow
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:ab59dc79e0b0b27268e56c52f29426cbd07d00fc210c9b51b657022993c07259
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - DADA2
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# taxonomic-classification-assignment

## Summary

Assign taxonomy to denoised representative sequences (ASVs) produced by QIIME 2 DADA2 or Deblur denoising within the Tourmaline Snakemake workflow, using one of the supported QIIME 2 classifiers and a configured reference database (e.g. SILVA 138.1 or PR2).

## When to use

Use when a Tourmaline run has produced a feature table and representative-sequence artifacts from DADA2/Deblur denoising and taxonomy must be assigned to the ASVs as the next workflow step before diversity analyses.

## When NOT to use

- Input is raw FASTQ reads that have not yet been denoised — run the DADA2/Deblur denoising step (qaqc/repseqs) first
- The workflow run is Tourmaline 2, where taxonomy should be invoked via the tourmaline.sh script with --step taxonomy rather than standalone V1 pseudo-rules

## Inputs

- Representative sequences (ASVs) artifact from QIIME 2 DADA2 or Deblur denoising
- Reference database taxonomy artifacts (e.g. SILVA 138.1, PR2) defined in the Tourmaline config file
- Tourmaline configuration file specifying parameters and input files

## Outputs

- Taxonomy assignments artifact (taxonomy of representative sequences)
- Taxonomy visualizations linked in the run HTML report (report_dada2-pe_unfiltered.html)

## How to apply

After the denoising step (e.g. the dada2_pe_denoise Snakemake pseudo-rule) produces the feature table and representative-sequence artifacts, run the Tourmaline taxonomy step (e.g. dada2_pe_taxonomy_unfiltered in the V1 paired-end unfiltered workflow, or the 'taxonomy' step of tourmaline.sh in Tourmaline 2) to assign taxonomy to the representative sequences using one of the supported QIIME 2 classifiers. The classifier and reference database (e.g. database_name: silva-138_1 or PR2) are defined in the Tourmaline configuration file along with the sample metadata and manifest. Taxonomy assignment is a prerequisite for the downstream taxonomic, alpha, and beta diversity analyses, so the resulting taxonomy artifact must be validated before running the diversity rules. Verify completion by confirming taxonomy assignments exist in the run directory alongside the feature table, representative sequences, and the final HTML report.

## Related tools

- **QIIME 2** (Runs the taxonomy classifiers that assign taxonomy to the denoised representative sequences) — https://github.com/aomlomics/tourmaline
- **Tourmaline** (Snakemake-based workflow wrapping the QIIME 2 taxonomy step and generating the HTML report linking the taxonomy outputs) — https://github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system executing the taxonomy rule (e.g. dada2_pe_taxonomy_unfiltered)) — https://github.com/aomlomics/tourmaline
- **DADA2** (Upstream denoising step producing the representative sequences that taxonomy is assigned to) — https://github.com/benjjneb/dada2
- **QIIME 2 viewer** (Web-viewable inspection of taxonomy visualizations produced by the workflow)

## Examples

```
./tourmaline.sh --step taxonomy --configfile config_tourmaline.yaml --cores 8
```

## Evaluation signals

- The taxonomy rule (e.g. dada2_pe_taxonomy_unfiltered) completes without error in the Snakemake run directory
- Taxonomy assignment outputs are present in the run directory alongside the DADA2 feature table and representative sequences
- The final HTML report (report_dada2-pe_unfiltered.html) is generated and contains links to the taxonomy visualizations
- Taxonomy artifact was produced using one of the supported QIIME 2 classifiers with the configured reference database (silva-138_1 or PR2)

## Limitations

- Tourmaline 1 (V1 branch) and Tourmaline 2 differ in invocation: V1 uses individual Snakemake rules while Tourmaline 2 requires specifying the correct Snakefile or using tourmaline.sh --step taxonomy
- Reference database choice (e.g. SILVA 138.1 vs PR2) is fixed in the configuration file and must match the amplicon target; no built-in validation of database suitability is described

## Evidence

- [abstract] Taxonomy assignment to representative sequences as a workflow step: "assigns taxonomy to the resulting representative sequences"
- [methods] V1 branch reference for the legacy workflow: "V1 branch: https://github.com/aomlomics/tourmaline/tree/V1"
- [intro] HTML report links to web-viewable QIIME 2 visualizations: "Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files"
- [readme] Tourmaline 2 taxonomy step invocation via tourmaline.sh: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
- [abstract] Configuration file defines reference database, metadata, and manifest: "Starting from a configuration file that defines parameters and input files—a reference database, a sample metadata file, and a manifest or archive of FASTQ sequences"
