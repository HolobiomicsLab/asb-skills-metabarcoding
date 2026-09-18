---
name: greengenes-lineage-format-interpretation
description: Use when building a counts matrix whose features carry Greengenes 13_8
  or SILVA 138 lineage strings with missing labels at family/genus/species ranks,
  and feature rows must have fully populated rank fields.
when_to_use_negative:
- Reference lineages already contain explicit labels at every rank (e.g., RefSeq full-genome
  taxonomy), so no propagation of lowest labels is needed
- The input is a raw read-level taxonomic assignment (e.g., Kraken 2 per-read calls)
  rather than an OTU/feature table with lineage metadata
- Downstream analysis requires preserving the original missing-label (empty rank)
  representation rather than propagated lowest-taxonomy placeholders
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: QIIME 2
  role: Produces feature tables whose taxonomy metadata is parsed and propagated rank-by-rank
- name: Mothur
  role: Produces OTU counts with Greengenes-style taxonomy strings parsed by this
    skill
- name: PathoScope 2
  role: Profiler whose per-feature lineages (including custom-configured Greengenes
    indices) are normalized by lineage propagation
- name: Kraken 2
  role: Profiler against Greengenes (built with kraken2-build --special greengenes)
    whose feature lineages are propagated
- name: MetaScope
  role: Package providing download_refseq() for obtaining up-to-date RefSeq reference
    libraries used alongside propagated lineages
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:ae94baf7bbf163a86008cbf0ca09c9b218b6e19d85de5f494162bf58680f699e
schema_version: 0.2.0
metadata:
  license_tier: open
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
yes_rate: 0.1111111111111111
n_claims: 9
---

# Greengenes lineage format interpretation

## Summary

Parses Greengenes-style rank-prefixed lineage strings (e.g., o_Bacillales) rank-by-rank and propagates the deepest available taxonomic label into higher-rank fields, recording the taxonomic best hit (available granularity) per feature. This makes OTU/feature taxonomy metadata complete and comparable across profiler-by-reference-database combinations.

## When to use

Use when building a counts matrix whose features carry Greengenes 13_8 or SILVA 138 lineage strings with missing labels at family/genus/species ranks, and feature rows must have fully populated rank fields.

## When NOT to use

- Reference lineages already contain explicit labels at every rank (e.g., RefSeq full-genome taxonomy), so no propagation of lowest labels is needed
- The input is a raw read-level taxonomic assignment (e.g., Kraken 2 per-read calls) rather than an OTU/feature table with lineage metadata
- Downstream analysis requires preserving the original missing-label (empty rank) representation rather than propagated lowest-taxonomy placeholders

## Inputs

- Per-profiler counts matrices from QIIME 2, Mothur, PathoScope 2, and Kraken 2 runs
- Reference-library taxonomy lineages from Greengenes 13_8, SILVA 138, and RefSeq
- Greengenes-format taxonomy files (e.g., gg_13_8_99 taxonomy from the Mothur website)

## Outputs

- taxonomic_best_hits.csv: one row per feature with all rank fields populated and an explicit best-hit rank annotation per database
- Propagated lineage strings preserving Greengenes rank prefixes (k__, p__, c__, o__, f__, g__, s__)

## How to apply

Load the per-profiler counts matrices produced by QIIME 2, Mothur, PathoScope 2, and Kraken 2 against SILVA 138, Greengenes 13_8, and RefSeq reference libraries, and parse each feature's lineage string rank-by-rank to find the deepest rank carrying a non-empty label. Copy that deepest label into every higher-rank field (e.g., an order-only Bacillales hit fills family, genus, and species fields with o_Bacillales), preserving Greengenes-style rank prefixes where present. For each feature and each database, record the deepest rank reached as the taxonomic best hit / available granularity annotation. Assemble propagated lineages and best-hit granularity across all profiler-by-database combinations into taxonomic_best_hits.csv.

## Related tools

- **QIIME 2** (Produces feature tables whose taxonomy metadata is parsed and propagated rank-by-rank)
- **Mothur** (Produces OTU counts with Greengenes-style taxonomy strings parsed by this skill)
- **PathoScope 2** (Profiler whose per-feature lineages (including custom-configured Greengenes indices) are normalized by lineage propagation)
- **Kraken 2** (Profiler against Greengenes (built with kraken2-build --special greengenes) whose feature lineages are propagated)
- **MetaScope** (Package providing download_refseq() for obtaining up-to-date RefSeq reference libraries used alongside propagated lineages) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- Every feature row has all rank fields populated with no empty values above the deepest available label
- Each row carries an explicit best-hit rank annotation (available granularity) per database
- Greengenes-style rank prefixes (e.g., o_ prefix for order-level labels) are preserved in propagated fields
- The propagated lineage of a feature never claims a more specific rank than the deepest non-empty label in the original reference lineage
- Output table taxonomic_best_hits.csv covers all profiler-by-database combinations (QIIME 2/Mothur/PathoScope 2/Kraken 2 x SILVA 138/Greengenes 13_8/RefSeq)

## Limitations

- Propagation records available granularity rather than true assignments: an order-only Bacillales hit is written into genus/species fields as o_Bacillales, so lower-rank fields do not represent genuine genus/species calls
- Reference libraries retrieved from public repositories may contain contamination requiring additional filtering before taxonomy parsing (e.g., RefSeq entries needing quality control)
- Greengenes 13_8 was the most recent Greengenes release at the time of the study; the newer Greengenes2 (2022) may use a different lineage format

## Evidence

- [discussion] PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe: "PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe"
- [readme] kraken2-build greengenes: "kraken2-build --db ${DBNAME} --special greengenes --threads 1"
- [readme] Mothur Greengenes indices source: "Mothur Greengenes indices were obtained from the [Mothur website.](https://mothur.org/wiki/silva_reference_files)"
- [methods] refseq download function: "Up-to-date RefSeq indices may be obtained by using the `download_refseq()` function from the [MetaScope package]"
