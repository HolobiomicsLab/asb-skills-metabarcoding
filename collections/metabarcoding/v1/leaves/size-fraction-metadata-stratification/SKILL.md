---
name: size-fraction-metadata-stratification
description: Use when per-sample QIIME 2 outputs (group significance .qzv, taxa barplots
  .qzv) must be grouped by a metadata column such as size fraction to compare richness
  or family dominance between 5.0-um and 0.22-um samples.
when_to_use_negative:
- Samples do not carry a size-fraction (or comparable grouping) metadata column, so
  no stratification variable exists to join on.
- The analysis targets a single pooled group rather than a between-fraction comparison
  (e.g., pure QC of denoising or primer trimming in the qaqc step).
- Input is already collapsed/aggregated per group — re-stratifying at the sample level
  would double-count or misattribute samples.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: QIIME 2
  role: Produces the .qzv visualizations and feature tables whose per-sample values
    are extracted and stratified by metadata
  repo: https://github.com/aomlomics/tourmaline
- name: QIIME 2 viewer
  role: Web-viewable inspection of taxonomy.qzv and taxa_barplot.qzv to check family-level
    relative abundances per size fraction
  repo: https://github.com/aomlomics/tourmaline
- name: Python
  role: Artifact API / pandas parsing of .qzv files and join of per-sample values
    with the sample metadata file
  repo: https://github.com/aomlomics/tourmaline
- name: Jupyter notebooks
  role: R- and Python-based notebooks that consume QIIME 2 artifact files for downstream
    stratified analysis
  repo: https://github.com/aomlomics/tourmaline
- name: Tourmaline
  role: 'Workflow that generated the unfiltered DADA2-PE outputs (config_03_taxonomy.yaml,
    SILVA 138.1, collapse_taxalevel: 7) consumed by this skill'
  repo: https://github.com/aomlomics/tourmaline
merged_aliases:
- size-fraction-metadata-stratified-abundance-analysis
merged_alias_records:
- alias: size-fraction-metadata-stratified-abundance-analysis
  slug: size-fraction-metadata-stratified-abundance-analysis
  cosine_score: 0.8422
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:7badcbc31507b09daf3ef27d91ef9e8b3ad6c552e7eb7020133d9a3ffc1cf57a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Jupyter notebooks
  - Python
  - QIIME 2
  - QIIME 2 viewer
  - Tourmaline
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# size-fraction-metadata-stratification

## Summary

Stratifies per-sample amplicon results (observed features, family-level abundances) by size-fraction sample metadata (5.0-μm vs 0.22-μm) in the Lake Erie 16S Tourmaline test dataset, enabling group-wise comparisons of alpha diversity and taxonomic dominance.

## When to use

Use when per-sample QIIME 2 outputs (group significance .qzv, taxa barplots .qzv) must be grouped by a metadata column such as size fraction to compare richness or family dominance between 5.0-um and 0.22-um samples.

## When NOT to use

- Samples do not carry a size-fraction (or comparable grouping) metadata column, so no stratification variable exists to join on.
- The analysis targets a single pooled group rather than a between-fraction comparison (e.g., pure QC of denoising or primer trimming in the qaqc step).
- Input is already collapsed/aggregated per group — re-stratifying at the sample level would double-count or misattribute samples.

## Inputs

- observed_features_group_significance.qzv (alpha group significance visualization from unfiltered diversity outputs)
- alpha_rarefaction.qzv (rarefaction visualization, plot_diversity: true, alpha_max_depth 500, core_sampling_depth 500)
- taxonomy.qzv and taxa_barplot.qzv (naive-Bayes classifier against SILVA 138.1, config_03_taxonomy.yaml)
- sample metadata file (00-data/metadata.tsv) with size-fraction column (5.0-um vs 0.22-um)
- feature table / representative-sequence taxonomy assignments from the unfiltered DADA2-PE Tourmaline test run

## Outputs

- Per-sample observed-features table stratified by size fraction
- Family-level relative abundance table grouped by 5.0-um vs 0.22-um strata
- Compact verification TSV (group medians, direction of difference, plateau depth, pass/fail flags) e.g., alpha_diversity_verification.tsv

## How to apply

Parse the QIIME 2 visualization (e.g., observed_features_group_significance.qzv from the unfiltered DADA2-PE run) with the QIIME 2 Artifact API / pandas to extract per-sample values, then join them to the sample metadata file (00-data/metadata.tsv) on sample ID to assign each sample to its size fraction. For alpha diversity, compute group-level summaries of observed features for the 5.0-um vs 0.22-um strata and check direction of difference against the expected finding (5.0-um richer, saturated at ~350 sequences/sample from alpha_rarefaction.qzv generated with alpha_max_depth 500 and core_sampling_depth 500). For taxonomy, collapse the feature table at family level (collapse_taxalevel: 7), compute per-group relative abundances, and stratify by the same metadata column to identify dominant families per fraction. The rationale is that Tourmaline's QIIME 2 artifacts carry sample metadata that can be fed directly into Python- or R-based Jupyter notebooks, so no re-import is needed.

## Related tools

- **QIIME 2** (Produces the .qzv visualizations and feature tables whose per-sample values are extracted and stratified by metadata) — https://github.com/aomlomics/tourmaline
- **QIIME 2 viewer** (Web-viewable inspection of taxonomy.qzv and taxa_barplot.qzv to check family-level relative abundances per size fraction) — https://github.com/aomlomics/tourmaline
- **Python** (Artifact API / pandas parsing of .qzv files and join of per-sample values with the sample metadata file) — https://github.com/aomlomics/tourmaline
- **Jupyter notebooks** (R- and Python-based notebooks that consume QIIME 2 artifact files for downstream stratified analysis) — https://github.com/aomlomics/tourmaline
- **Tourmaline** (Workflow that generated the unfiltered DADA2-PE outputs (config_03_taxonomy.yaml, SILVA 138.1, collapse_taxalevel: 7) consumed by this skill) — https://github.com/aomlomics/tourmaline

## Examples

```
import pandas as pd; meta = pd.read_csv('00-data/metadata.tsv', sep='\t'); stratified = observed_features.join(meta.set_index('sampleid')[['size_fraction']], how='inner')
```

## Evaluation signals

- Group medians show the 5.0-um fraction has higher observed features than the 0.22-um fraction in the group-significance results.
- Rarefaction curves from alpha_rarefaction.qzv plateau at approximately 350 sequences per sample (within the 500-depth rarefaction run).
- The 5.0-um stratum's top families are Sporichthyaceae and SAR11 Clade III; the 0.22-um stratum's top family is Cyanobiaceae (chloroplast-dominated, Microcystaceae lower).
- Every sample in the joined table is assigned to exactly one size fraction via the metadata join (no dropped or duplicated sample IDs).
- Verification TSV contains explicit pass/fail flags matching each expected dominance/saturation pattern.

## Limitations

- Saturation was only assessed up to alpha_max_depth 500 with core_sampling_depth 500; conclusions about the ~350 sequences/sample plateau do not extend beyond that range.
- The comparison relies on the unfiltered DADA2-PE test run; applying the same stratification to filtered runs (taxonomy, length, abundance, prevalence, or ID filtering) may change observed features and family rankings.
- The skill is specific to the Lake Erie surface-water 16S dataset and its sample metadata (size-fraction categories 5.0-um and 0.22-um); other datasets need their own metadata grouping column.

## Evidence

- [abstract] Lake Erie dataset framing: "We ran Tourmaline on a 16S ribosomal RNA amplicon data set from Lake Erie surface water"
- [intro] Artifacts feed notebooks for downstream analysis: "QIIME 2 artifact files can be fed directly into provided Python- and R-based Jupyter notebooks"
- [methods] Family-level collapse parameter grounding: "repseq_min_length: 0
repseq_max_length: 0"
- [readme] Tourmaline 2 as the producing workflow: "Tourmaline 2 is an amplicon sequence processing workflow for Illumina sequence data that uses [QIIME 2](https://qiime2.org) and the software packages it wraps."
