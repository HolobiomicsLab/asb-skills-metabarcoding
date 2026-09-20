---
name: shannon-entropy-computation
description: Use when quantifying how narrowly a feature (exact sequence, OTU, genus,
  or phylogenetic subtree) is distributed across environment categories — e.g. testing
  whether EMP Deblur tag sequences are more habitat-specific than higher taxonomic
  groups.
when_to_use_negative:
- The input table lacks environment-category metadata in the mapping file, so no environment
  distribution can be computed per feature
- The goal is within-sample alpha diversity rather than across-environment specificity
  of individual features
- Features are clustered OTUs from heterogeneous, non-standardized protocols where
  exact Deblur sequences cannot be followed across studies
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Python
  role: Implements the Shannon entropy computation and entropy-versus-resolution plots
    over merged BIOM/mapping data
- name: Pandas
  role: Merges BIOM-derived tables with mapping metadata and aggregates counts by
    taxonomic level for per-unit entropy
- name: QIIME1
  role: Merges the BIOM tables and mapping files before entropy computation
- name: Deblur
  role: Provides the exact 90-bp V4 tag sequences whose per-sequence environment distributions
    are the finest-resolution units of the entropy analysis
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:37105bae3f30a9d69b7228df42154b8324cd109c7d5d048080f561abc14d8f83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cuttlefishh/deblur
  tools:
  - Deblur
  - Pandas
  - Python
  - QIIME1
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# shannon-entropy-computation

## Summary

Computes Shannon entropy H = -Σ(p_i · log2 p_i) over the distribution of environment categories observed per feature unit (exact Deblur tag sequences, genera, or phylogenetic subtrees) to quantify habitat specificity. In the EMP 16S Release 1 analysis, entropy dropped sharply at the tips of the tree, showing that environment specificity is best captured by individual 90-bp V4 tag sequences rather than 97%-identity OTUs or genera.

## When to use

Use when quantifying how narrowly a feature (exact sequence, OTU, genus, or phylogenetic subtree) is distributed across environment categories — e.g. testing whether EMP Deblur tag sequences are more habitat-specific than higher taxonomic groups.

## When NOT to use

- The input table lacks environment-category metadata in the mapping file, so no environment distribution can be computed per feature
- The goal is within-sample alpha diversity rather than across-environment specificity of individual features
- Features are clustered OTUs from heterogeneous, non-standardized protocols where exact Deblur sequences cannot be followed across studies

## Inputs

- EMP 16S Release 1 Deblur tag-sequence BIOM table (exact 90-bp V4 tag sequences)
- Closed-reference OTU BIOM tables (e.g. picked against Greengenes 13.8)
- Merged quality-filtered mapping files with environment-category metadata
- Taxonomic assignment / phylogenetic tree for collapsing counts to genera, higher taxa, and subtrees

## Outputs

- Per-unit Shannon entropy values (in bits) for tag sequences, genera, higher taxonomic groups, and phylogenetic subtrees
- Plot of entropy versus unit type marking the 97% identity species-threshold resolution
- Comparison demonstrating tag-sequence-level environment specificity

## How to apply

Merge the BIOM tables with the quality-filtered mapping files using QIIME1 or QIIME2 tooling, then rarefy to 5,000 observations per sample, exclude sequences annotated as chloroplast, and apply mc2 filtering (minimum sequence count in an OTU equals 2). For each exact Deblur tag sequence (90-bp 16S rRNA V4 region, starting after primer 515f), compute H = -Σ(p_i · log2 p_i) in Python/Pandas over the proportions of environment categories observed for that sequence. Collapse counts to genera and higher taxonomic groups and to phylogenetic subtrees, recomputing entropy per aggregated unit. Compare entropy across resolution levels: the expected EMP pattern is a gradual decrease toward finer resolution with a sharp drop at the tips (tag sequences below the 97% identity species threshold show low entropy, i.e. high environment specificity, whereas genera and above are more evenly distributed across environments).

## Related tools

- **Python** (Implements the Shannon entropy computation and entropy-versus-resolution plots over merged BIOM/mapping data)
- **Pandas** (Merges BIOM-derived tables with mapping metadata and aggregates counts by taxonomic level for per-unit entropy)
- **QIIME1** (Merges the BIOM tables and mapping files before entropy computation)
- **Deblur** (Provides the exact 90-bp V4 tag sequences whose per-sequence environment distributions are the finest-resolution units of the entropy analysis) — https://github.com/cuttlefishh/deblur

## Examples

```
import numpy as np; H = -np.sum(p * np.log2(p))  # p = proportions of environment categories observed for each Deblur tag sequence
```

## Evaluation signals

- Every entropy value satisfies 0 ≤ H ≤ log2(number of environment categories)
- Entropy values for individual tag sequences are sharply lower than for any higher taxonomic group or phylogenetic subtree, matching the Fig. 4 pattern
- Counts were rarefied to 5,000 observations per sample and chloroplast-annotated sequences were excluded before statistics
- mc2 filtering applied: only OTUs with minimum sequence count ≥ 2 retained
- Entropy decreases gradually from higher taxa down but drops sharply below the 97% identity species threshold

## Limitations

- Rarefaction to 5,000 observations per sample discards samples with fewer reads and caps per-sample counts
- Entropy magnitudes depend on the chosen environment-category scheme; comparing across different category granularities is not meaningful
- Exact tag sequences are 90-bp V4 amplicons, so specificity conclusions are tied to that region and to Deblur processing rather than full-length 16S

## Evidence

- [supplementary] Rarefaction threshold used before entropy statistics: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Chloroplast exclusion filter: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] Definition of the mc2 filter parameter: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] Tag sequences are the exact-sequence features analyzed: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] Tool for merging BIOM tables and mapping files: [evidence span withheld — anchor status: not_found]
- [abstract] Rationale for exact sequences over clustered OTUs: "Coordinated protocols and new analytical methods, particularly the use of exact sequences instead of clustered operational taxonomic units, enable bacterial and archaeal ribosomal RNA gene sequences"
