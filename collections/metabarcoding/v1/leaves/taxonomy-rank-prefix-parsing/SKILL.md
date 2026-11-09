---
name: taxonomy-rank-prefix-parsing
description: Use when a counts matrix built from a profiler/reference pairing has
  features with incomplete taxonomies (e.g., an order-level-only lineage) and downstream
  genus- or species-level benchmarking requires every rank field populated with an
  explicit -hit granularity annotation.
when_to_use_negative:
- Input features already carry non-empty labels at every rank (k__ through s__), so
  no propagation is needed
- The goal is a genuine genus/species-level assignment rather than a propagated best-hit
  label — propagated fields overstate the available granularity (e.g., an order-only
  hit does not become a true species call)
- The profiler output does not expose a traceable taxonomic hierarchy, so lineage
  strings cannot be parsed rank-by-rank
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: Source of per-profiler counts matrices whose feature lineage strings are parsed
    and propagated
  repo: github.com/aubreyodom/16SBenchmarking
- name: Mothur
  role: Source of per-profiler counts matrices with traceable taxonomic hierarchies
    used for rank parsing
  repo: github.com/aubreyodom/16SBenchmarking
- name: PathoScope 2
  role: Source of per-profiler counts matrices with taxonomic hierarchy tracing for
    lineage propagation
  repo: github.com/aubreyodom/16SBenchmarking
- name: Kraken 2
  role: Source of per-profiler counts matrices against SILVA, Greengenes, and RefSeq/Kraken
    2 Standard libraries for lineage parsing
  repo: github.com/aubreyodom/16SBenchmarking
- name: MetaScope
  role: Package supplying download_refseq() for building the up-to-date RefSeq indices
    underlying the parsed lineages
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:d5c83167f2dfddb3bf6e21d79c24bad7d93674615a3d9eb631ce60951ec7a7ef
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# taxonomy-rank-prefix-parsing

## Summary

Parses per-feature lineage strings from 16S profiling outputs (QIIME 2, Mothur, PathoScope 2, Kraken 2 against SILVA 138, Greengenes 13_8, and RefSeq) rank-by-rank, propagates the deepest available taxonomic label into every higher-rank field, and annotates each feature with the deepest rank reached (taxonomic best hit). This enables consistent family/genus/species comparison across profiler-by-reference-library combinations.

## When to use

Use when a counts matrix built from a profiler/reference pairing has features with incomplete taxonomies (e.g., an order-level-only lineage) and downstream genus- or species-level benchmarking requires every rank field populated with an explicit best-hit granularity annotation.

## When NOT to use

- Input features already carry non-empty labels at every rank (k__ through s__), so no propagation is needed
- The goal is a genuine genus/species-level assignment rather than a propagated best-hit label — propagated fields overstate the available granularity (e.g., an order-only hit does not become a true species call)
- The profiler output does not expose a traceable taxonomic hierarchy, so lineage strings cannot be parsed rank-by-rank

## Inputs

- Per-profiler OTU/feature counts matrices from QIIME 2, Mothur, PathoScope 2, and Kraken 2 (one per reference-library pairing)
- Feature lineage strings from SILVA 138, Greengenes 13_8, and RefSeq reference libraries, parsed rank-by-rank

## Outputs

- taxonomic_best_hits.csv — a single table of propagated lineages and per-feature, per-database deepest-rank (best-hit granularity) annotations across all profiler-by-database combinations

## How to apply

Load the per-profiler counts matrices produced by QIIME 2, Mothur, PathoScope 2, and Kraken 2 against the SILVA 138, Greengenes 13_8, and RefSeq reference libraries, then parse each feature's lineage string rank-by-rank to locate the deepest rank carrying a non-empty label. Copy that deepest label into every higher-rank field — e.g., an order-only Bacillales hit fills the family, genus, and species fields with o_Bacillales — preserving Greengenes-style rank prefixes (k__, p__, c__, o__, f__, g__, s__) where present. For each feature and each database, record the deepest rank reached as the taxonomic best hit / available granularity annotation. Assemble the propagated lineages and best-hit granularity across all profiler-by-database combinations into a single table written to taxonomic_best_hits.csv. The rationale is that PathoScope, QIIME 2, and Mothur allow tracing back the taxonomic hierarchy of a given microbe, so propagation along the database's taxonomic path preserves available granularity without fabricating finer assignments.

## Related tools

- **QIIME 2** (Source of per-profiler counts matrices whose feature lineage strings are parsed and propagated) — github.com/aubreyodom/16SBenchmarking
- **Mothur** (Source of per-profiler counts matrices with traceable taxonomic hierarchies used for rank parsing) — github.com/aubreyodom/16SBenchmarking
- **PathoScope 2** (Source of per-profiler counts matrices with taxonomic hierarchy tracing for lineage propagation) — github.com/aubreyodom/16SBenchmarking
- **Kraken 2** (Source of per-profiler counts matrices against SILVA, Greengenes, and RefSeq/Kraken 2 Standard libraries for lineage parsing) — github.com/aubreyodom/16SBenchmarking
- **MetaScope** (Package supplying download_refseq() for building the up-to-date RefSeq indices underlying the parsed lineages) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
combined.to_csv('taxonomic_best_hits.csv')  # propagated lineages + per-database best-hit rank across profiler x reference combinations
```

## Evaluation signals

- Every feature row in taxonomic_best_hits.csv has all rank fields populated with no empty values above the deepest available label
- Each row carries an explicit best-hit rank annotation per database (the deepest rank reached)
- Greengenes-style rank prefixes (e.g., o_Bacillales) are preserved in the propagated fields where the source lineage used them
- Row coverage matches the full profiler-by-database grid (QIIME 2, Mothur, PathoScope 2, Kraken 2 × SILVA 138, Greengenes 13_8, RefSeq)
- Spot-check a known order-only lineage: higher-rank fields are filled with the identical prefixed label, and no finer-rank label is invented

## Limitations

- Propagated labels at family/genus/species are placeholders carrying the available granularity — they do not indicate true assignments at those ranks, so downstream genus/species accuracy metrics reflect the best-hit rank
- The quality of propagation depends on the reference database; the paper found SILVA and RefSeq/Kraken 2 Standard libraries superior to Greengenes, so Greengenes lineages may terminate at coarser ranks
- Mock communities provide only a semblance of ground truth, so propagated labels cannot be validated against a perfect expected taxonomy

## Evidence

- [other] Deepest-label propagation across ranks: [evidence span withheld — anchor status: not_found]
- [discussion] Rationale: tools expose traceable taxonomic hierarchy: "PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe"
- [readme] Reference libraries supplying the parsed lineages: "Mothur Greengenes indices were obtained from the Mothur website."
- [other] Output artifact grounding: [evidence span withheld — anchor status: not_found]
