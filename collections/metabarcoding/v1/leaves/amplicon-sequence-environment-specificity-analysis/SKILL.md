---
name: amplicon-sequence-environment-specificity-analysis
description: Use when you have a large multi-study amplicon feature table (e.g., Deblur
  tag sequences or OTU BIOM tables) plus sample metadata with environment categories,
  and you need to determine at which phylogenetic resolution habitat specificity is
  captured.
when_to_use_negative:
- The goal is alpha/beta diversity comparison rather than habitat-specificity of individual
  sequences
- The input lacks per-sample environment-category metadata, so entropy over environment
  distributions cannot be computed
- Amplicons are non-16S regions (e.g., ITS or 18S) where the V4 tag-sequence framing
  and 97% species-threshold interpretation do not apply
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0610
- http://edamontology.org/topic_3174
tools:
- name: Python
  role: Used with Pandas to merge BIOM tables and mapping files, compute Shannon entropy
    of environment distributions, and plot entropy by unit type
  repo: https://github.com/biocore/emp/tree/master/methods
- name: Pandas
  role: DataFrame manipulation for merging metadata with BIOM tables and aggregating
    counts to taxonomic levels
- name: Deblur
  role: Produces the exact-sequence tag sequences (90-bp V4) whose environment distributions
    are analyzed
  repo: https://github.com/cuttlefishh/deblur
- name: QIIME1
  role: Merges the BIOM tables and mapping files; closed-reference OTU picking context
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/nature24621
    title: A communal catalogue reveals Earth’s multiscale microbial diversity
  source_content_hash: sha256:ac4077479060b67bcc3b51ffc3eae240d29e7b93eb1609a092d018f02e35c6b5
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Deblur
  - Pandas
  - Python
  - QIIME1
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0610
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# amplicon-sequence-environment-specificity-analysis

## Summary

Quantifies at which taxonomic/phylogenetic resolution environment specificity is best captured by computing Shannon entropy H = -sum(p_i * log2 p_i) of environment-category distributions per exact amplicon tag sequence and comparing it to entropy after collapsing counts to genera, higher taxa, and phylogenetic subtrees, as in the EMP 16S Release 1 meta-analysis.

## When to use

Use when you have a large multi-study amplicon feature table (e.g., Deblur tag sequences or OTU BIOM tables) plus sample metadata with environment categories, and you need to determine at which phylogenetic resolution habitat specificity is best captured.

## When NOT to use

- The goal is alpha/beta diversity comparison rather than habitat-specificity of individual sequences
- The input lacks per-sample environment-category metadata, so entropy over environment distributions cannot be computed
- Amplicons are non-16S regions (e.g., ITS or 18S) where the V4 tag-sequence framing and 97% species-threshold interpretation do not apply

## Inputs

- Deblur tag-sequence BIOM table (90-bp V4 sequences, EMP 16S Release 1)
- Closed-reference OTU BIOM tables (e.g., against Greengenes 13.8)
- Merged quality-filtered mapping files with environment-category metadata
- Reference taxonomy for collapsing tag sequences to genera and phylogenetic subtrees

## Outputs

- Per-unit Shannon entropy values (H in bits) for tag sequences, higher taxa, and phylogenetic subtrees
- Plot of entropy versus taxonomic unit type annotated at the 97% identity species threshold
- Summary of which resolution (exact tag sequence) best captures environment specificity

## How to apply

Download the EMP 16S Release 1 Deblur tag-sequence BIOM table, closed-reference OTU BIOM tables, and merged quality-filtered mapping files from ftp://ftp.microbio.me/emp/release1, then use QIIME1 (or QIIME2) to merge the BIOM tables and mapping files in Python/Pandas. Apply the standard EMP filters: rarefy to 5,000 observations per sample, exclude sequences annotated as chloroplast, and apply mc2 filtering (minimum sequence count in an OTU equals 2). For each exact Deblur tag sequence (90-bp 16S rRNA V4 region starting after primer 515f), compute the Shannon entropy of its environment-category distribution; then collapse counts to genera and higher taxonomic groups and to phylogenetic subtrees and recompute entropy per aggregated unit. Plot entropy versus unit type marking the 97% identity species threshold; the expected result is that entropy drops sharply at the tips of the tree, so exact tag sequences show low entropy (high environment specificity) whereas genera and higher levels are more evenly distributed across environments.

## Related tools

- **Python** (Used with Pandas to merge BIOM tables and mapping files, compute Shannon entropy of environment distributions, and plot entropy by unit type) — https://github.com/biocore/emp/tree/master/methods
- **Pandas** (DataFrame manipulation for merging metadata with BIOM tables and aggregating counts to taxonomic levels)
- **Deblur** (Produces the exact-sequence tag sequences (90-bp V4) whose environment distributions are analyzed) — https://github.com/cuttlefishh/deblur
- **QIIME1** (Merges the BIOM tables and mapping files; closed-reference OTU picking context)

## Examples

```
redbiom search metadata 'envo_biome' | redbiom fetch samples --context Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506 --output emp.biom
```

## Evaluation signals

- Entropy values for individual tag sequences are sharply lower than for any genus, higher taxon, or phylogenetic subtree, matching the Fig. 4 pattern
- Every computed entropy value lies in the valid range 0 <= H <= log2(number of environment categories)
- Filters were applied as in the paper: 5,000 observations per sample after rarefaction, chloroplast sequences excluded, mc2 (minimum sequence count of 2) enforced
- Tag sequences analyzed are 90-bp V4 sequences starting after primer 515f from the Deblur algorithm
- Genera and higher taxonomic levels show relatively even environment distributions (higher entropy) compared to tips of the tree

## Limitations

- Conclusions hold for the EMP 16S Release 1 data (~25,000 samples, 97 studies) with Deblur exact sequences; results may differ for clustered OTUs or other datasets
- Rarefying to 5,000 observations per sample discards samples below that depth, reducing the effective sample set
- The sharp specificity at exact tag sequences means results are below the 97% identity threshold defining microbial species and do not generalize to species-level assignments

## Evidence

- [supplementary] Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm.: "Tag sequences are 90-bp 16S rRNA gene sequences (V4 region, starting after primer 515f) from the Deblur algorithm."
- [supplementary] Samples were rarefied to 5,000 observations per sample.: "Samples were rarefied to 5,000 observations per sample."
- [supplementary] Sequences annotated as chloroplast were excluded before statistics were computed.: "Sequences annotated as chloroplast were excluded before statistics were computed."
- [methods] mc2 refers to minimum sequence count in an OTU to be included equals to 2.: "mc2 refers to minimum sequence count in an OTU to be included equals to 2."
- [methods] You can then use QIIME1 or QIIME2 to merge the BIOM tables and mapping files.: [evidence span withheld — anchor status: not_found]
- [abstract] Coordinated protocols and use of exact sequences instead of clustered OTUs enable bacterial and archaeal rRNA gene sequences to be followed across multiple studies: [evidence span withheld — anchor status: not_found]
