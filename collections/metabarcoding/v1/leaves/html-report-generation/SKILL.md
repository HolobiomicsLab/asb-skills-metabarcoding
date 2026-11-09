---
name: html-report-generation
description: Use when a completed Tourmaline workflow run (denoising, taxonomy, diversity)
  needs to be consolidated into one HTML report with linked, web-viewable QIIME 2
  visualizations for experts and nonexperts.
when_to_use_negative:
- The workflow run has not produced its analysis outputs yet — the report rule summarizes
  and links existing denoising, taxonomy, and diversity outputs rather than computing
  them
- You need interactive downstream analysis in Python or R — use the provided Jupyter
  notebooks that consume QIIME 2 artifact files instead of the static HTML report
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0654
tools:
- name: Tourmaline
  role: Workflow whose report rules (e.g. dada2_pe_report_unfiltered) build the HTML
    report for each run
  repo: https://github.com/aomlomics/tourmaline
- name: Snakemake
  role: Workflow management system that executes the report pseudo-rule within the
    Tourmaline Snakefile
  repo: https://snakemake.readthedocs.io/en/stable/
- name: QIIME 2
  role: Produces the .qzv visualization files that the HTML report links to
- name: QIIME 2 viewer
  role: Alternative web tool for viewing the interactive visualizations linked from
    the report
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:9c2be88cabb7c6f6298aaa6e651ae25a00f766144165bd4f131c5720a7286a49
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Snakemake
  - Tourmaline
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# html-report-generation

## Summary

Generates a single shareable HTML report (e.g. report_dada2-pe_unfiltered.html) that summarizes a Tourmaline/QIIME 2 amplicon run — metadata, inputs, outputs, parameters — and links to web-viewable QIIME 2 visualization files (.qzv) that can also be opened in QIIME 2 viewer.

## When to use

Use when a completed Tourmaline workflow run (denoising, taxonomy, diversity) needs to be consolidated into one HTML report with linked, web-viewable QIIME 2 visualizations for experts and nonexperts.

## When NOT to use

- The workflow run has not produced its analysis outputs yet — the report rule summarizes and links existing denoising, taxonomy, and diversity outputs rather than computing them
- You need interactive downstream analysis in Python or R — use the provided Jupyter notebooks that consume QIIME 2 artifact files instead of the static HTML report

## Inputs

- Tourmaline run directory containing QIIME 2 artifacts (feature table, representative sequences, taxonomy) and visualizations (.qzv)
- Sample metadata file
- Tourmaline configuration file defining parameters (reference database, manifest or FASTQ archive, trimming/filter settings)

## Outputs

- Single comprehensive HTML report (e.g. report_dada2-pe_unfiltered.html) summarizing parameters, metadata, inputs, and outputs
- Links to web-viewable QIIME 2 visualization files (.qzv)

## How to apply

After running the analysis rules of a Tourmaline workflow (e.g. dada2_pe_denoise, dada2_pe_taxonomy_unfiltered, dada2_pe_diversity_unfiltered on the Lake Erie 16S test dataset), execute the report rule for the corresponding run — for the unfiltered paired-end DADA2 workflow this is the snakemake pseudo-rule dada2_pe_report_unfiltered. The rule assembles a summary of the run's parameters, sample metadata, input and output files, and embeds or links web-viewable QIIME 2 visualization files (.qzv) so they can be inspected via the HTML report or QIIME 2 viewer. In Tourmaline 2 the same report step can also be triggered through ./tourmaline.sh rather than by invoking the snakemake rule directly. Validation is done by confirming the run directory contains report_dada2-pe_unfiltered.html (or the report matching the run's denoiser/read mode) alongside the feature table, representative sequences, taxonomy, and diversity outputs, and that the report's links resolve to viewable visualizations.

## Related tools

- **Tourmaline** (Workflow whose report rules (e.g. dada2_pe_report_unfiltered) build the HTML report for each run) — https://github.com/aomlomics/tourmaline
- **Snakemake** (Workflow management system that executes the report pseudo-rule within the Tourmaline Snakefile) — https://snakemake.readthedocs.io/en/stable/
- **QIIME 2** (Produces the .qzv visualization files that the HTML report links to)
- **QIIME 2 viewer** (Alternative web tool for viewing the interactive visualizations linked from the report)

## Examples

```
snakemake --use-conda --cores 8 dada2_pe_report_unfiltered
```

## Evaluation signals

- The run directory contains the expected report file (e.g. report_dada2-pe_unfiltered.html) after the report rule completes
- The HTML report includes a summary of parameters, metadata, inputs, and outputs for the run
- Links in the report resolve to web-viewable QIIME 2 visualization files (.qzv)
- The report corresponds to the specific run variant (e.g. unfiltered paired-end DADA2) with its taxonomy and diversity outputs present
- The same report content is independently viewable via QIIME 2 viewer

## Limitations

- The report is static — interactive exploration of the underlying QIIME 2 artifacts requires QIIME 2 viewer or the provided Jupyter notebooks
- In Tourmaline 2 the workflow must be run from the tourmaline directory (or with the correct Snakefile specified) for the report rule to resolve its inputs
- No changelog is available for tracking report-format changes (missing changelog noted in the discussion)

## Evidence

- [other] Running the final command, dada2_pe_report_unfiltered, creates a single comprehensive HTML report (report_dada2-pe_unfiltered.html) summarizing the parameters, metadata, inputs, outputs, and visualizations of the workflow run.: [evidence span withheld — anchor status: not_found]
- [intro] Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files: "Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files"
- [abstract] generates an HTML report summarizing and linking to the output files: "generates an HTML report summarizing and linking to the output files"
- [readme] Instead of interacting with Snakemake rules directly, the main way to run Tourmaline 2 is through the `tourmaline.sh` script.: "Instead of interacting with Snakemake rules directly, the main way to run Tourmaline 2 is through the `tourmaline.sh` script."
- [abstract] easily view interactive visualizations through the HTML report, QIIME 2 viewer: "easily view interactive visualizations through the HTML report, QIIME 2 viewer"
