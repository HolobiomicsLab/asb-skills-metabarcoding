---
name: microbiome-environment-stratification
description: Use when an EMP 16S Release 1 BIOM table and mapping file must be analyzed
  per environment type — e.g., computing NODF nestedness separately for non-saline,
  saline, animal, and plant subsets of a 2,000-sample subset rather than only globally.
when_to_use_negative:
- The analysis question requires abundance-based or beta-diversity comparison (e.g.,
  weighted UniFrac) rather than presence-absence nestedness — this skill binarizes
  the abundance table and discards count information.
- The mapping file lacks environment-category metadata, so the non-saline/saline/animal/plant
  strata cannot be defined.
- The goal is functional (e.g., metabolomic) rather than taxonomic composition stratification
  of EMP samples.
edam_topics: []
tools:
- name: Python
  role: 'Scripting environment for the meta-analysis: loading the EMP 16S Release
    1 BIOM table and mapping file and computing NODF, z-scores, and permutation p-values'
  repo: github.com/biocore/emp
- name: Pandas
  role: Reading the BIOM and mapping-file tables, applying the mc2 and non-chloroplast
    filters, building binary presence-absence matrices, and writing nodf_results.csv
- name: redbiom
  role: Command-line querying of the Qiita-hosted EMP catalogue to retrieve sample
    data and metadata by context before environment stratification
  repo: github.com/biocore/redbiom
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:68de9c69dc8410288253ff8419c100af90fefe302f598fe7bcd7f5ee4c0abb1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Pandas
  - Python
  - redbiom
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# microbiome-environment-stratification

## Summary

Stratifies EMP 16S Release 1 samples by environment category (non-saline, saline, animal, plant) and computes taxonomic nestedness (NODF) within each stratum against an equiprobable-rows/fixed-columns null model, showing communities are significantly nested (NODF = 0.841 ± 0.018) with phyla gained but not lost as richness increases.

## When to use

Use when an EMP 16S Release 1 BIOM table and mapping file must be analyzed per environment type — e.g., computing NODF nestedness separately for non-saline, saline, animal, and plant subsets of a 2,000-sample subset rather than only globally.

## When NOT to use

- The analysis question requires abundance-based or beta-diversity comparison (e.g., weighted UniFrac) rather than presence-absence nestedness — this skill binarizes the abundance table and discards count information.
- The mapping file lacks environment-category metadata, so the non-saline/saline/animal/plant strata cannot be defined.
- The goal is functional (e.g., metabolomic) rather than taxonomic composition stratification of EMP samples.

## Inputs

- EMP 16S Release 1 BIOM abundance table for the 2,000-sample subset (16S V4 tag sequences)
- EMP merged mapping/metadata file with environment category labels (e.g., non-saline, saline, animal, plant)

## Outputs

- nodf_results.csv containing NODF statistics, z-scores, and permutation p-values overall, per taxonomic level (phylum, class, order), and per environment subset
- Binary presence-absence matrices at phylum, class, and order levels, per environment stratum

## How to apply

Load the EMP 16S Release 1 BIOM table and mapping file for the 2,000-sample subset into Python with pandas, restricting to non-chloroplast taxa and applying the mc2 (minimum sequence count = 2) filter. Convert abundance tables to binary presence-absence matrices aggregated at the phylum, class, and order levels, then subset each matrix by the mapping file's environment categories. Compute the nestedness metric NODF (Nestedness based on Overlap and Decreasing Fill) on each taxonomic-level matrix and on per-environment subsets (non-saline, saline, animal, plant). Generate null distributions by randomizing each matrix under an equiprobable-rows/fixed-columns null model with many permutations, then compute empirical NODF, z-scores, and permutation p-values. Decision point: acceptance is judged by the overall NODF falling within 0.841 ± 0.018 and all per-environment values being significant at P < 0.05 relative to the null distribution; write all NODF statistics and p-values to nodf_results.csv.

## Related tools

- **Python** (Scripting environment for the meta-analysis: loading the EMP 16S Release 1 BIOM table and mapping file and computing NODF, z-scores, and permutation p-values) — github.com/biocore/emp
- **Pandas** (Reading the BIOM and mapping-file tables, applying the mc2 and non-chloroplast filters, building binary presence-absence matrices, and writing nodf_results.csv)
- **redbiom** (Command-line querying of the Qiita-hosted EMP catalogue to retrieve sample data and metadata by context before environment stratification) — github.com/biocore/redbiom

## Examples

```
import pandas as pd; nodf_results = pd.DataFrame({'env': ['non_saline', 'saline', 'animal', 'plant'], 'nodf': [0.811, 0.787, 0.788, 0.860]}); nodf_results.to_csv('nodf_results.csv')
```

## Evaluation signals

- Computed overall NODF falls within 0.841 ± 0.018 with P < 0.0001 versus the null model.
- Per-environment NODF values match the reported values: 0.811 non-saline, 0.787 saline, 0.788 animal, 0.860 plant, each significant at P < 0.05.
- Presence-absence matrices are successfully produced at all three aggregation levels (phylum, class, order) for each environment subset.
- Null distributions derive from an equiprobable-rows/fixed-columns randomization with many permutations, and empirical z-scores/p-values are reported for every matrix.
- All NODF statistics and p-values are written to nodf_results.csv.

## Limitations

- Nestedness is computed on binary presence-absence matrices, so relative-abundance information is not used.
- Results depend on the chosen null model (equiprobable-rows/fixed-columns); a different randomization scheme would change p-values.
- The analysis applies only to the filtered 2,000-sample subset: chloroplast-annotated sequences excluded and mc2 (minimum sequence count = 2) enforced, so findings may not transfer to unfiltered tables.

## Evidence

- [methods] Compute the nestedness metric NODF (Nestedness based on Overlap and Decreasing Fill) on each taxonomic-level matrix and on per-environment subsets (non-saline, saline, animal, plant).: [evidence span withheld — anchor status: not_found]
- [methods] Generate null distributions by randomizing each matrix under an equiprobable-rows/fixed-columns null model with many permutations.: [evidence span withheld — anchor status: not_found]
- [other] NODF = 0.841 ± 0.018 (95% CI) and P < 0.0001 versus the null model.: [evidence span withheld — anchor status: not_found]
- [supplementary] Sequences annotated as chloroplast were excluded before statistics were computed.: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [readme] EMP 16S Release 1, a meta-analysis of the first 97 16S rRNA amplicon studies, has been published: "EMP 16S Release 1, a meta-analysis of the first 97 16S rRNA amplicon studies, has been published"
