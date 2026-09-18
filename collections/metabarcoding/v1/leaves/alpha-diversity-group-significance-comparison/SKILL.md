---
name: alpha-diversity-group-significance-comparison
description: Use when a Tourmaline/QIIME 2 run has produced diversity outputs (.qzv)
  and the question is whether one metadata group (e.g., size fraction) has higher
  observed features than another, and at what sampling depth richness saturates.
when_to_use_negative:
- 'The run is still pre-diversity: no .qzv diversity outputs exist yet (denoising
  with DADA2/Deblur or taxonomy assignment has not completed).'
- The comparison target is beta diversity (between-sample community structure) rather
  than within-sample richness.
- Input is already a per-sample feature table exported from QIIME 2 — the group-significance
  .qzv parsing step is unnecessary.
edam_operation: http://edamontology.org/operation_0538
tools:
- name: Tourmaline
  role: produces the diversity .qzv outputs and config parameters (alpha_max_depth,
    core_sampling_depth) consumed by this verification skill
  repo: github.com/aomlomics/tourmaline
- name: QIIME 2
  role: generates the alpha diversity group-significance and rarefaction visualizations
    parsed here
- name: Python
  role: parses .qzv files via the QIIME 2 Artifact API and joins richness values with
    metadata using pandas
- name: Jupyter notebooks
  role: interactive environment for consuming QIIME 2 artifact files during the verification
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/gigascience/giac066
    title: 'Tourmaline: A containerized workflow for rapid and iterable amplicon sequence
      analysis using QIIME 2 and Snakemake'
  source_content_hash: sha256:29341f0defb74758f8609698d5a3181a4f2c8a5ce00243a2aee79d41bdc0777b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aomlomics/tourmaline
  tools:
  - Jupyter notebooks
  - Python
  - QIIME 2
  - Tourmaline
  edam_operation: http://edamontology.org/operation_0538
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# alpha-diversity-group-significance-comparison

## Summary

Verify within-sample (alpha) diversity differences between sample groups in a Tourmaline/QIIME 2 amplicon run by parsing the observed-features group-significance .qzv and the alpha-rarefaction .qzv, then stratifying per-sample richness against metadata (e.g., 5.0-μm vs 0.22-μm size fractions) and checking rarefaction plateau depth (~350 sequences/sample in the Lake Erie 16S test run).

## When to use

Use when a Tourmaline/QIIME 2 run has produced diversity outputs (.qzv) and the question is whether one metadata group (e.g., size fraction) has higher observed features than another, and at what sampling depth richness saturates.

## When NOT to use

- The run is still pre-diversity: no .qzv diversity outputs exist yet (denoising with DADA2/Deblur or taxonomy assignment has not completed).
- The comparison target is beta diversity (between-sample community structure) rather than within-sample richness.
- Input is already a per-sample feature table exported from QIIME 2 — the group-significance .qzv parsing step is unnecessary.

## Inputs

- observed_features_group_significance.qzv (QIIME 2 group significance visualization from the unfiltered Tourmaline diversity outputs)
- alpha_rarefaction.qzv (rarefaction visualization generated with alpha_max_depth 500, core_sampling_depth 500, plot_diversity true)
- sample metadata file (e.g., 00-data/metadata.tsv) with size-fraction grouping column

## Outputs

- alpha_diversity_verification.tsv with group medians, direction of difference, plateau depth, and pass/fail flags

## How to apply

Parse the unfiltered run's observed_features_group_significance.qzv with the QIIME 2 Artifact/Visualization API or pandas to extract per-sample observed-features values. Join those values with the sample metadata file (e.g., 00-data/metadata.tsv) to split samples into groups (5.0-um vs 0.22-um size fraction) and compare richness direction and group medians. Separately parse alpha_rarefaction.qzv — generated with plot_diversity: true, alpha_max_depth 500 and core_sampling_depth 500 in the Tourmaline config — and identify the depth at which observed-features curves plateau (expected ~350 sequences/sample). Write a compact verification table (group medians, direction of difference, plateau depth, pass/fail flags) to alpha_diversity_verification.tsv. Validation passes if the 5.0-μm fraction shows higher observed features than the 0.22-μm fraction and curves level off by ~350 sequences per sample.

## Related tools

- **Tourmaline** (produces the diversity .qzv outputs and config parameters (alpha_max_depth, core_sampling_depth) consumed by this verification skill) — github.com/aomlomics/tourmaline
- **QIIME 2** (generates the alpha diversity group-significance and rarefaction visualizations parsed here)
- **Python** (parses .qzv files via the QIIME 2 Artifact API and joins richness values with metadata using pandas)
- **Jupyter notebooks** (interactive environment for consuming QIIME 2 artifact files during the verification)

## Examples

```
from qiime2 import Visualization; v = Visualization.load('observed_features_group_significance.qzv'); v.export('obs_out')
```

## Evaluation signals

- 5.0-μm fraction shows higher observed features than the 0.22-μm fraction across the group-significance results
- Rarefaction curves level off by approximately 350 sequences per sample
- alpha_diversity_verification.tsv contains group medians, direction of difference, plateau depth, and pass/fail flags
- Per-sample extracted values correctly join to the size-fraction column in 00-data/metadata.tsv (no unmatched sample IDs)

## Limitations

- Rarefaction plateau depth is config-dependent (alpha_max_depth/core_sampling_depth = 500 here); a different config changes the expected saturation point.
- The expected result (5.0-μm > 0.22-μm, plateau ~350 seq/sample) is specific to the Lake Erie 16S test dataset; other datasets will have different directions and plateaus.
- Group significance .qzv reflects the unfiltered run; filtered runs (repseq_min_* parameters) will change observed-features values.

## Evidence

- [abstract] Tourmaline runs alpha and beta diversity analyses producing QIIME 2 visualizations parsed here: "performs analyses of taxonomic, alpha, and beta diversity"
- [intro] QIIME 2 artifacts feed Python Jupyter notebooks for downstream verification: "QIIME 2 artifact files can be fed directly into provided Python- and R-based Jupyter notebooks"
- [abstract] Tourmaline invokes QIIME 2 via Snakemake from a configuration file: "We developed Tourmaline, a Python-based workflow that implements QIIME 2 and is built using the Snakemake workflow management system"
- [readme] Tourmaline runs are executed step-wise via tourmaline.sh with a config file: "./tourmaline.sh --step [qaqc,repseqs,taxonomy] --configfile [config1,config2,config3] --cores N"
