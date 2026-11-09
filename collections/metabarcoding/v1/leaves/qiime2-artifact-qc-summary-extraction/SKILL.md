---
name: qiime2-artifact-qc-summary-extraction
description: Use when comparing multiple Tourmaline/QIIME 2 runs (e.g., a DADA2 truncation-length
  sweep) and you need per-run representative sequence length distributions and taxonomy-assigned
  read counts aggregated into a summary table.
when_to_use_negative:
- Input is already an aggregated feature table without per-run stats directories —
  the skill requires the Tourmaline run directory structure
- A single run with a fixed parameter set is being analyzed and no cross-run comparison
  or length-distribution summary is needed
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Tourmaline
  role: Produces the per-run directory structure ([run_name]-repseqs/stats/) and HTML
    report whose QC summaries are extracted and compared
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2
  role: Underlying analysis package whose artifacts (.qza) contain the sequence-length
    reports and taxonomy visualizations being summarized
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2 viewer
  role: Web-based viewer for interactively inspecting the QIIME 2 visualization files
    linked from the Tourmaline HTML report
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:f25a0f143c8f24b9a3942371bca941c053f48890983d874d0e1fa9c111486869
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - QIIME 2
  - QIIME 2 viewer
  - Tourmaline
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# qiime2-artifact-qc-summary-extraction

## Summary

Extracts QC summary statistics — such as representative sequence length distributions and per-run truncation stats — from Tourmaline repseqs/stats/ output directories and QIIME 2 artifacts, enabling cross-run comparison of parameter settings like DADA2 truncation lengths.

## When to use

Use when comparing multiple Tourmaline/QIIME 2 runs (e.g., a DADA2 truncation-length sweep) and you need per-run representative sequence length distributions and taxonomy-assigned read counts aggregated into a summary table.

## When NOT to use

- Input is already an aggregated feature table without per-run stats directories — the skill requires the Tourmaline run directory structure
- A single run with a fixed parameter set is being analyzed and no cross-run comparison or length-distribution summary is needed

## Inputs

- Tourmaline repseqs/stats/ directories from multiple runs (one per parameter setting)
- QIIME 2 sequence artifacts (.qza) with sequence-length reports
- Taxonomy step outputs with read counts per taxon (e.g., Eukaryota assigned via PR2)

## Outputs

- Aggregated summary table (e.g., truncation_sweep_summary.csv) with one row per parameter setting
- Per-run representative sequence length distribution summaries
- Target-taxon read counts per run (e.g., Eukaryota reads)

## How to apply

After running the Tourmaline repseqs step for each parameter setting (each with a distinct run_name), navigate to each [run_name]-repseqs/stats/ directory and extract the sequence-length reports describing the representative sequence length distribution. Pair each length distribution with the read counts assigned to the target taxon (e.g., Eukaryota via the PR2 database) from the corresponding taxonomy step. Aggregate the per-run distributions and read counts into a single table (e.g., truncation_sweep_summary.csv) with one row per parameter setting. Interactive inspection of the underlying QIIME 2 artifacts can be done via the run's HTML report or QIIME 2 viewer links, which Tourmaline generates for every run.

## Related tools

- **Tourmaline** (Produces the per-run directory structure ([run_name]-repseqs/stats/) and HTML report whose QC summaries are extracted and compared) — https://github.com/aomlomics/tourmaline
- **QIIME 2** (Underlying analysis package whose artifacts (.qza) contain the sequence-length reports and taxonomy visualizations being summarized) — https://github.com/aomlomics/tourmaline
- **QIIME 2 viewer** (Web-based viewer for interactively inspecting the QIIME 2 visualization files linked from the Tourmaline HTML report)

## Examples

```
./tourmaline.sh --step qaqc --configfile config_01_qaqc.yaml --cores N  # then inspect [run_name]-repseqs/stats/ sequence-length reports per truncation setting
```

## Evaluation signals

- The summary table contains exactly one row per truncation/parameter setting in the sweep (e.g., the four settings {(100,100), (150,150), (200,200), (250,250)})
- Each row includes both a sequence length distribution summary and target-taxon (Eukaryota) assigned read counts
- Resulting trends across settings reproduce the patterns reported in Supplementary Fig. S4A and S4B
- Extracted values are traceable to files under the correct [run_name]-repseqs/stats/ path, matching the run_name used in each config
- QIIME 2 artifact provenance confirms each summary derives from a dada2pe repseqs run with the intended trunc_len_f/trunc_len_r settings

## Limitations

- Extraction depends on Tourmaline's defined directory structure and run_name conventions; non-Tourmaline QIIME 2 outputs require different paths
- Reads truncated too aggressively can distort length distributions and reduce taxonomic assignment, so trends must be interpreted jointly with quality scores
- No changelog was found for the repository, so version-specific changes to output paths may be undocumented

## Evidence

- [other] Extract the representative sequence length distributions from the sequence-length reports under each [run_name]-repseqs/stats/ directory.: [evidence span withheld — anchor status: not_found]
- [other] Aggregate the per-run length distributions and Eukaryota read counts into a single table and save as truncation_sweep_summary.csv.: [evidence span withheld — anchor status: not_found]
- [intro] different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script: "different runs that utilize different parameters (e.g., DADA2 truncation lengths) can be easily compared, facilitated by a helper script"
- [intro] Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files: "Every Tourmaline run produces an HTML report containing a summary of metadata and outputs, with links to web-viewable QIIME 2 visualization files"
