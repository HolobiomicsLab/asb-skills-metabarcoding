---
name: phylogenetic-subtree-entropy-aggregation
description: Use when you need to determine at which taxonomic or phylogenetic resolution
  environment (habitat) specificity is captured in a large multi-study 16S dataset
  such as EMP 16S Release 1.
when_to_use_negative:
- Input is a clustered-OTU-only table with no exact Deblur tag sequences — the sharp
  entropy drop at exact-sequence (below 97% identity) resolution cannot be observed
- Samples lack reliable environment-category metadata, since entropy is computed over
  the distribution of environment categories per feature
- The goal is beta-diversity or community-composition comparison rather than resolution-level
  habitat-specificity assessment
tools:
- name: Python
  role: Used with Pandas to merge BIOM tables and mapping files, compute per-sequence
    Shannon entropy over environment categories, and plot entropy across resolutions
- name: Pandas
  role: Data-frame handling of merged BIOM tables and mapping metadata during entropy
    computation and taxonomic collapsing
- name: QIIME1
  role: Merging the BIOM tables and mapping files (biom tooling) before the entropy
    analysis
- name: Deblur
  role: Source of the exact 90-bp V4 tag sequences whose per-sequence environment
    entropy is computed
  repo: https://github.com/cuttlefishh/deblur
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:1023a7d504beec2a00f50df6fb75cebf2b82a1a5ecb12551424ab79ce0161045
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
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# phylogenetic-subtree-entropy-aggregation

## Summary

Quantifies environment specificity across phylogenetic resolutions by computing Shannon entropy of environment-category distributions per exact Deblur 16S tag sequence and re-computing it after collapsing counts to genera, higher taxa, and phylogenetic subtrees, revealing that specificity is sharpest at the exact-sequence level.

## When to use

Use when you need to determine at which taxonomic or phylogenetic resolution environment (habitat) specificity is best captured in a large multi-study 16S dataset such as EMP 16S Release 1.

## When NOT to use

- Input is a clustered-OTU-only table with no exact Deblur tag sequences — the sharp entropy drop at exact-sequence (below 97% identity) resolution cannot be observed
- Samples lack reliable environment-category metadata, since entropy is computed over the distribution of environment categories per feature
- The goal is beta-diversity or community-composition comparison rather than resolution-level habitat-specificity assessment

## Inputs

- EMP 16S Release 1 Deblur tag-sequence BIOM table (90-bp V4 exact sequences)
- Closed-reference OTU BIOM tables (Greengenes 13.8)
- Merged quality-filtered mapping files with environment category metadata

## Outputs

- Shannon entropy of environment distribution per Deblur tag sequence, per genus/higher taxon, and per phylogenetic subtree
- Plot of entropy versus unit type (tag sequence vs. higher taxa vs. subtrees) with the 97% identity species-threshold marked

## How to apply

Download the EMP 16S Release 1 Deblur tag-sequence BIOM table, closed-reference OTU BIOM tables, and merged quality-filtered mapping files from ftp://ftp.microbio.me/emp/release1, then use QIIME1 (or QIIME2) biom tooling with Python/Pandas to merge the BIOM tables and mapping files. Apply the paper's filtering regime: rarefy to 5,000 observations per sample, exclude sequences annotated as chloroplast, and apply mc2 filtering (minimum sequence count in an OTU equals 2). For each exact 90-bp V4 Deblur tag sequence, compute Shannon entropy H = -sum(p_i * log2 p_i) over the distribution of environment categories observed for that sequence. Collapse sequence counts to genera and higher taxonomic groups and to phylogenetic subtrees (closed-reference against Greengenes 13.8), recompute entropy per aggregated unit, and plot entropy versus unit type in Python, marking the 97% identity species-threshold resolution. The expected result is that entropy drops sharply at the tips of the tree: individual tag sequences below the 97% species threshold show low entropy (high environment specificity) while genera and higher taxa are more evenly distributed across environments.

## Related tools

- **Python** (Used with Pandas to merge BIOM tables and mapping files, compute per-sequence Shannon entropy over environment categories, and plot entropy across resolutions)
- **Pandas** (Data-frame handling of merged BIOM tables and mapping metadata during entropy computation and taxonomic collapsing)
- **QIIME1** (Merging the BIOM tables and mapping files (biom tooling) before the entropy analysis)
- **Deblur** (Source of the exact 90-bp V4 tag sequences whose per-sequence environment entropy is computed) — https://github.com/cuttlefishh/deblur

## Examples

```
import pandas as pd, numpy as np; p = counts_by_env / counts_by_env.sum(); H = -float((p * np.log2(p)).sum())  # per Deblur tag sequence over environment categories
```

## Evaluation signals

- Every entropy value lies between 0 and log2(number of environment categories)
- Entropy values for individual Deblur tag sequences are sharply lower than for any higher taxonomic group or phylogenetic subtree (reproducing the Fig. 4 pattern)
- Filtering regime was applied identically: 5,000 observations per sample rarefaction, chloroplast sequences excluded, and mc2 (minimum sequence count = 2) enforced
- Collapsed units (genus and above, subtrees) show gradually higher entropy than the tips, with the drop occurring below the 97% identity species threshold

## Limitations

- Results depend on the specific filtering regime (5,000-read rarefaction, chloroplast exclusion, mc2); other thresholds may shift entropy values
- Environment specificity is assessed only over the environment categories recorded in the merged EMP mapping files; coarse or inconsistent metadata weakens the entropy contrast
- The sharp drop is specific to exact tag sequences (Deblur V4); clustered OTU tables at 97% identity will not reproduce the tip-level signal

## Evidence

- [supplementary] rarefaction: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] chloroplast exclusion: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 filter: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [supplementary] tag sequences: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [methods] merge tables: [evidence span withheld — anchor status: not_found]
