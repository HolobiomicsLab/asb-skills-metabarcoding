---
name: qiime2-artifact-generation
description: Use when raw demultiplexed Illumina amplicon FASTQ data (e.g. a Lake
  Erie 16S rRNA dataset) plus a Tourmaline configuration file must be converted into
  QIIME 2 feature-table, representative-sequence, taxonomy, and diversity artifacts.
when_to_use_negative:
- The data are single-end reads destined for Deblur denoising, which Tourmaline supports
  as the single-end alternative to DADA2 paired-end
- Input is already a denoised QIIME 2 feature table or representative sequence artifact
  (Tourmaline 2 does allow starting mid-workflow with externally generated data, so
  skip denoising rules)
- The goal is filtering representative sequences (taxonomy, length, abundance, prevalence,
  or ID) rather than generating the initial unfiltered artifact set
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3697
tools:
- name: Tourmaline
  role: Python/Snakemake workflow that wraps QIIME 2 commands into the pseudo-rules
    (dada2_pe_denoise, dada2_pe_taxonomy_unfiltered, dada2_pe_diversity_unfiltered,
    dada2_pe_report_unfiltered) and produces the artifacts
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Runs the DADA2 paired-end denoising algorithm and taxonomy/diversity plugins
    that generate the artifacts and .qzv visualizations
- name: Snakemake
  role: Workflow management system that manages commands, inputs, and outputs of each
    rule; invoked per-rule with the correct Snakefile
  repo: https://snakemake.readthedocs.io/en/stable/
- name: DADA2
  role: Denoising algorithm invoked by QIIME 2 to produce the feature table and representative
    sequences
  repo: https://github.com/benjjneb/dada2
- name: QIIME 2 viewer
  role: Used to view the interactive .qzv visualizations linked from the generated
    HTML report
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:3cbc1e6bcaf5483a74b1bfa268654d5b77040e1d7e7e3e51ae87e5937315f8ad
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# qiime2-artifact-generation

## Summary

Generate QIIME 2 artifacts (feature table, representative sequences, taxonomy assignments, diversity results, and an HTML report) from Illumina amplicon FASTQ by running Tourmaline's Snakemake rules, which wrap QIIME 2's DADA2 paired-end denoising and downstream plugins.

## When to use

Use when raw demultiplexed Illumina amplicon FASTQ data (e.g. a Lake Erie 16S rRNA dataset) plus a Tourmaline configuration file must be converted into QIIME 2 feature-table, representative-sequence, taxonomy, and diversity artifacts.

## When NOT to use

- The data are single-end reads destined for Deblur denoising, which Tourmaline supports as the single-end alternative to DADA2 paired-end
- Input is already a denoised QIIME 2 feature table or representative sequence artifact (Tourmaline 2 does allow starting mid-workflow with externally generated data, so skip denoising rules)
- The goal is filtering representative sequences (taxonomy, length, abundance, prevalence, or ID) rather than generating the initial unfiltered artifact set

## Inputs

- Demultiplexed paired-end Illumina FASTQ files (manifest or FASTQ archive)
- Tourmaline configuration file (reference database such as SILVA 138.1 or PR2, sample metadata file, denoising parameters)
- Sample metadata file
- QIIME 2 taxonomic classifier (one of the supported classifiers)

## Outputs

- QIIME 2 feature table artifact (from dada2_pe_denoise)
- QIIME 2 representative sequence artifact
- Taxonomy assignments for unfiltered representative sequences
- Taxonomic, alpha, and beta diversity analysis outputs
- HTML report report_dada2-pe_unfiltered.html summarizing parameters, metadata, inputs, outputs, and visualizations

## How to apply

Start from a configuration file that defines parameters and input files—a reference database (e.g. SILVA 138.1 or PR2), a sample metadata file, and a manifest or archive of FASTQ sequences—and activate the Snakemake environment (e.g. snakemake-tour2) plus the bundled QIIME 2 environment. Run the Snakemake pseudo-rule dada2_pe_denoise, which invokes QIIME 2's DADA2 paired-end denoising to produce the feature table and representative sequence artifacts; trimming parameters can be determined automatically from quality scores. Then run dada2_pe_taxonomy_unfiltered to assign taxonomy to the unfiltered representative sequences with a supported QIIME 2 classifier, followed by dada2_pe_diversity_unfiltered to compute taxonomic, alpha, and beta diversity analyses on the unfiltered feature table. Finally run dada2_pe_report_unfiltered, which produces a single comprehensive HTML report (report_dada2-pe_unfiltered.html) summarizing parameters, metadata, inputs, outputs, and visualizations with links to web-viewable QIIME 2 visualization files. Each rule lives in its own Snakefile (Tourmaline 2 splits qaqc, repseqs, and taxonomy into three steps), so the correct Snakefile must be specified when invoking an individual rule; the tourmaline.sh wrapper script can run one or more steps with a chosen config file and core count.

## Related tools

- **Tourmaline** (Python/Snakemake workflow that wraps QIIME 2 commands into the pseudo-rules (dada2_pe_denoise, dada2_pe_taxonomy_unfiltered, dada2_pe_diversity_unfiltered, dada2_pe_report_unfiltered) and produces the artifacts) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Runs the DADA2 paired-end denoising algorithm and taxonomy/diversity plugins that generate the artifacts and .qzv visualizations)
- **Snakemake** (Workflow management system that manages commands, inputs, and outputs of each rule; invoked per-rule with the correct Snakefile) — https://snakemake.readthedocs.io/en/stable/
- **DADA2** (Denoising algorithm invoked by QIIME 2 to produce the feature table and representative sequences) — https://github.com/benjjneb/dada2
- **QIIME 2 viewer** (Used to view the interactive .qzv visualizations linked from the generated HTML report)

## Examples

```
snakemake dada2_pe_denoise dada2_pe_taxonomy_unfiltered dada2_pe_diversity_unfiltered dada2_pe_report_unfiltered
```

## Evaluation signals

- The run directory contains the DADA2 feature table, representative sequences, taxonomy assignments, diversity outputs, and the linked HTML report as described in the Tourmaline documentation
- The final rule dada2_pe_report_unfiltered creates exactly one comprehensive HTML file, report_dada2-pe_unfiltered.html, whose links open valid QIIME 2 visualization files
- Each Snakemake rule (dada2_pe_denoise, dada2_pe_taxonomy_unfiltered, dada2_pe_diversity_unfiltered, dada2_pe_report_unfiltered) completes without error in the activated Snakemake + QIIME 2 conda environments
- The HTML report summarizes the parameters, metadata, inputs, and outputs of the workflow run, matching the Tourmaline claim that every run produces such a report
- Provenance in the QIIME 2 artifacts traces back to the configuration-file-defined reference database, sample metadata, and FASTQ manifest

## Limitations

- Rule names such as dada2_pe_report_unfiltered belong to the Tourmaline V1 branch (V1 branch of the repository); Tourmaline 2 instead runs steps via tourmaline.sh and requires specifying the correct Snakefile per step
- The tourmaline.sh script must be run from inside the tourmaline directory, although output destinations can be set elsewhere
- The workflow targets Illumina amplicon sequence data and runs natively on Linux and macOS or via a Docker container

## Evidence

- [abstract] it uses QIIME 2 to run either the DADA2 or Deblur denoising algorithm: "it uses QIIME 2 to run either the DADA2 or Deblur denoising algorithm"
- [intro] Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files: "Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files"
- [abstract] Starting from a configuration file that defines parameters and input files—a reference database, a sample metadata file, and a manifest or archive of FASTQ sequences: "Starting from a configuration file that defines parameters and input files—a reference database, a sample metadata file, and a manifest or archive of FASTQ sequences"
- [readme] You can still run individual snakemake rules as before. Each of the three steps (explained more below) has its own Snakefile, so you must specify the correct snakefile when running an individual rule.: "You can still run individual snakemake rules as before. Each of the three steps (explained more below) has its own Snakefile, so you must specify the correct snakefile when running an individual rule."
- [abstract] performs analyses of taxonomic, alpha, and beta diversity: "performs analyses of taxonomic, alpha, and beta diversity"
