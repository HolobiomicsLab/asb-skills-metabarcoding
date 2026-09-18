---
name: per-database-taxonomic-best-hit-annotation
description: Use when per-profiler counts matrices (QIIME 2, Mothur, PathoScope 2,
  Kraken 2 against SILVA 138, Greengenes 13_8, RefSeq) contain features with empty
  family/genus/species lineage fields that must be filled before genus- or species-level
  benchmarking.
when_to_use_negative:
- The feature taxonomy is already complete at all target ranks in every database pairing
  — no propagation is needed
- The task is raw taxonomic classification itself (running Kraken 2, PathoScope 2,
  QIIME 2, or Mothur) rather than post-hoc normalization of already-produced feature
  taxonomies
- Inputs are whole-genome metagenomic assemblies or non-16S profiling outputs where
  the Greengenes/SILVA 16S lineage structures do not apply
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank
    for propagation
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank
    for propagation
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank
    for propagation
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank
    for propagation
  repo: github.com/aubreyodom/16SBenchmarking
- name: MetaScope
  role: Provides download_refseq() for obtaining/updating the RefSeq reference libraries
    whose lineage granularity is annotated
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:db7131cf3dfdc9773864d5f6c9b62cf2cf73cf93ba7deee1bb0268b876ebca6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/aubreyodom/16SBenchmarking
  tools:
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 9
---

# per-database-taxonomic-best-hit-annotation

## Summary

Completes partial taxonomic lineages from 16S profiler outputs by propagating the deepest available label (taxonomic best hit) into every higher-rank field, and annotates each feature with the deepest rank reached per reference database. This enables fair genus/species-level comparison across profiler-by-database combinations where reference lineages have uneven granularity.

## When to use

Use when per-profiler counts matrices (QIIME 2, Mothur, PathoScope 2, Kraken 2 against SILVA 138, Greengenes 13_8, RefSeq) contain features with empty family/genus/species lineage fields that must be filled before genus- or species-level benchmarking.

## When NOT to use

- The feature taxonomy is already complete at all target ranks in every database pairing — no propagation is needed
- The task is raw taxonomic classification itself (running Kraken 2, PathoScope 2, QIIME 2, or Mothur) rather than post-hoc normalization of already-produced feature taxonomies
- Inputs are whole-genome metagenomic assemblies or non-16S profiling outputs where the Greengenes/SILVA 16S lineage structures do not apply

## Inputs

- Per-profiler counts matrices from QIIME 2, Mothur, PathoScope 2, and Kraken 2 runs against SILVA 138, Greengenes 13_8, RefSeq, and Kraken 2 Standard reference libraries
- Reference-database lineage strings (Greengenes-style with k_/p_/c_/o_/f_/g_/s_ rank prefixes; SILVA and RefSeq lineage formats)

## Outputs

- taxonomic_best_hits.csv: one row per feature per database, with all rank fields populated (empty values replaced by the deepest propagated label) plus an explicit best-hit rank / available granularity annotation

## How to apply

Parse each feature's lineage string rank-by-rank (kingdom/phylum/class/order/family/genus/species) to locate the deepest rank carrying a non-empty label. Copy that deepest label into every higher-rank field — e.g., an order-only Bacillales hit fills family, genus, and species with o_Bacillales — preserving Greengenes-style rank prefixes (k_, p_, c_, o_, f_, g_, s_) where present in the source lineage. For each feature and each reference database (SILVA 138, Greengenes 13_8, RefSeq, Kraken 2 Standard), record the deepest rank reached as the taxonomic best hit / available granularity annotation. Rationale: the paper evaluated genus- and species-level accuracy across profiler/reference pairs, so features lacking explicit labels at those levels must still carry propagated, granularity-annotated assignments for a like-for-like comparison of sensitivity, specificity, and error score.

## Related tools

- **QIIME 2** (Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank for propagation) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank for propagation) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank for propagation) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Source of per-feature taxonomy counts matrices whose lineages are parsed rank-by-rank for propagation) — github.com/aubreyodom/16SBenchmarking
- **MetaScope** (Provides download_refseq() for obtaining/updating the RefSeq reference libraries whose lineage granularity is annotated) — https://compbiomed.github.io/metascope-docs/index.html

## Evaluation signals

- Every feature row has all rank fields populated — no empty values above the deepest available label in any lineage
- Each row carries an explicit best-hit rank annotation for each database, matching the deepest rank found in the original lineage string
- Greengenes-style rank prefixes (e.g., o_ for order) are preserved in propagated labels where they appear in the source database
- All profiler-by-database combinations (QIIME 2, Mothur, PathoScope 2, Kraken 2 × SILVA 138, Greengenes 13_8, RefSeq) are represented in a single taxonomic_best_hits.csv table

## Limitations

- Propagated labels record available granularity, not true identifications — commonly used 16S tools cannot produce high-quality genus- or species-level calls when the reference lineage is shallow
- Greengenes 13_8 (released August 2013) and SILVA 138.1 lineages differ in format and depth, so propagation behavior and best-hit ranks are database-dependent
- RefSeq libraries retrieved from public repositories may contain contamination, requiring additional filtering before lineage annotation

## Evidence

- [discussion] PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe: "PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe"
- [intro] we specifically analyzed genus- and species-level performance across pairs of profilers and libraries: "we specifically analyzed genus- and species-level performance across pairs of profilers and libraries"
- [abstract] the most-used bioinformatics tools cannot produce high quality genus-level or species-level taxonomic calls: "the most-used bioinformatics tools cannot produce high quality genus-level or species-level taxonomic calls"
- [readme] Kraken 2 Greengenes indices were obtained by running the following code: "Kraken 2 Greengenes indices were obtained by running the following code"
- [discussion] this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
