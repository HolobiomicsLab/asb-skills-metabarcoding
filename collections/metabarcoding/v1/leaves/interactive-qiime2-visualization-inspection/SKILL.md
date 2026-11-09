---
name: interactive-qiime2-visualization-inspection
description: Use when a Tourmaline/QIIME 2 run has produced .qzv visualization artifacts
  (taxonomy.qzv, taxa_barplot.qzv) and you need to visually verify or extract per-sample,
  family-level taxonomic composition, or when comparing runs with different parameters
  (e.g., DADA2-PE unfiltered vs filtered).
when_to_use_negative:
- You need programmatic/quantitative extraction of abundance tables rather than visual
  inspection — use the Jupyter notebooks that consume QIIME 2 artifact files instead
  of .qzv viewing.
- The input is raw FASTQ or an unprocessed feature table — taxonomy visualization
  requires the Tourmaline taxonomy step to have already run.
- You are working with a pipeline that does not emit QIIME 2 .qzv artifacts (e.g.,
  Anacapa or standalone Deblur output).
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2 viewer
  role: Web-based tool used to open and interactively explore taxonomy.qzv and taxa_barplot.qzv
    for family-level abundance inspection
- name: Tourmaline
  role: Produces the .qzv visualization artifacts and the HTML report that links to
    them during the taxonomy step of the workflow
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Generates the interactive visualizations (.qzv archive format) with built-in
    provenance tracking; also provides `qiime tools view` for local inspection
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:0cdd770729a82134c5f178f1205eb232a15a0c3aa7518ccedbc961203c9a19b2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Tourmaline
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# Interactive QIIME 2 Visualization Inspection

## Summary

Open and interrogate Tourmaline's QIIME 2 .qzv interactive visualizations (e.g., taxonomy.qzv, taxa_barplot.qzv) via QIIME 2 viewer to inspect family-level relative abundances and verify taxonomic composition findings, such as the dominance of Sporichthyaceae and SAR11 Clade III in the 5.0-μm fraction and Cyanobiaceae in the 0.22-μm fraction of the Lake Erie 16S rRNA test dataset.

## When to use

Use when a Tourmaline/QIIME 2 run has produced .qzv visualization artifacts (taxonomy.qzv, taxa_barplot.qzv) and you need to visually verify or extract per-sample, family-level taxonomic composition, or when comparing runs with different parameters (e.g., DADA2-PE unfiltered vs filtered).

## When NOT to use

- You need programmatic/quantitative extraction of abundance tables rather than visual inspection — use the Jupyter notebooks that consume QIIME 2 artifact files instead of .qzv viewing.
- The input is raw FASTQ or an unprocessed feature table — taxonomy visualization requires the Tourmaline taxonomy step to have already run.
- You are working with a pipeline that does not emit QIIME 2 .qzv artifacts (e.g., Anacapa or standalone Deblur output).

## Inputs

- taxonomy.qzv (QIIME 2 interactive visualization of representative-sequence taxonomy assignments)
- taxa_barplot.qzv (QIIME 2 interactive taxonomic bar plot visualization)
- Tourmaline run HTML report linking to the .qzv files
- Sample metadata with size-fraction categories (5.0-um vs 0.22-um)

## Outputs

- Visually confirmed family-level relative abundances per size fraction (e.g., Sporichthyaceae and SAR11 Clade III top in 5.0-um; Cyanobiaceae top in 0.22-um)
- Verified family-level abundances and pass/fail flags written to a TSV file

## How to apply

After the Tourmaline taxonomy step (config_03_taxonomy.yaml with classify_method naive-bayes against SILVA 138.1) completes, open the generated .qzv files — taxonomy.qzv and taxa_barplot.qzv — in QIIME 2 viewer (view.qiime2.org or `qiime tools view`). In taxa_barplot.qzv, set the taxonomic level to family (collapse_taxalevel: 7; taxa_ranks includes family) and group/compare samples by size-fraction metadata (5.0-um vs 0.22-um) to read relative abundances per family. Use these interactive views to confirm expected dominance patterns before writing family-level abundances and pass/fail flags to a TSV. Every Tourmaline run's HTML report links to these web-viewable QIIME 2 visualization files, so the report is the entry point for locating the correct .qzv artifacts from the run directory.

## Related tools

- **QIIME 2 viewer** (Web-based tool used to open and interactively explore taxonomy.qzv and taxa_barplot.qzv for family-level abundance inspection)
- **Tourmaline** (Produces the .qzv visualization artifacts and the HTML report that links to them during the taxonomy step of the workflow) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Generates the interactive visualizations (.qzv archive format) with built-in provenance tracking; also provides `qiime tools view` for local inspection)

## Examples

```
qiime tools view taxonomy.qzv; qiime tools view taxa_barplot.qzv
```

## Evaluation signals

- The taxa_barplot.qzv view at family level (taxa_ranks includes family, collapse_taxalevel: 7) shows Sporichthyaceae and SAR11 Clade III as the two most abundant families in the 5.0-um fraction.
- The 0.22-um fraction shows Cyanobiaceae as the most abundant family, with chloroplast sequences as its largest component and Microcystaceae less abundant.
- The output TSV contains relative abundances per family plus pass/fail verification flags for each expected dominance pattern.
- The .qzv files being inspected were produced by the unfiltered DADA2-PE test run using naive-bayes classification against SILVA 138.1, as documented in the run's provenance.
- The run's HTML report contains working links to the inspected QIIME 2 visualization files.

## Limitations

- Interactive .qzv inspection is visual and manual; quantitative confirmation requires exporting or consuming QIIME 2 artifacts programmatically (e.g., via the provided Jupyter notebooks).
- Findings are specific to the unfiltered DADA2-PE Tourmaline run; filtered runs (representative sequence filtering by taxonomy, length, abundance, prevalence, or ID) may shift family dominance patterns.
- Cyanobiaceae dominance in the 0.22-μm fraction is partly driven by chloroplast sequences, so chloroplast contamination must be considered when interpreting cyanobacterial abundance.

## Evidence

- [abstract] QIIME 2 viewer usage grounded in abstract: "showing its utility for parameter optimization and the ability to easily view interactive visualizations through the HTML report, QIIME 2 viewer"
- [intro] HTML report links to web-viewable visualizations: "Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files"
- [intro] QIIME 2 interactive visualizations and archive format: "QIIME 2, with its built-in provenance tracking, archive format, interactive visualizations, multiple interfaces including a Python API, and extensible plugin architecture"
- [intro] Jupyter notebooks for downstream consumption of artifacts: "QIIME 2 artifact files can be fed directly into provided Python- and R-based Jupyter notebooks"
