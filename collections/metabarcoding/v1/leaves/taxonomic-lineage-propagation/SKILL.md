---
name: taxonomic-lineage-propagation
description: Use when building counts matrices from profiler/reference pairings (QIIME
  2, Mothur, PathoScope 2, Kraken 2 vs. SILVA 138, Greengenes 13_8, RefSeq) and feature
  taxonomy strings contain empty rank fields above the deepest assigned label.
when_to_use_negative:
- Feature taxonomy metadata is already complete with every rank (kingdom through species)
  explicitly labeled — propagation would overwrite database-provided labels
- Amplicon pipeline outputs ground-truth species-level calls per feature; here the
  most-used 16S tools cannot produce high quality genus- or species-level calls, so
  propagation is only an approximation of granularity
- Reference lineages from a database with no rank-prefix conventions are being merged
  with Greengenes-prefixed lineages without preserving prefixes — stripping prefixes
  would break Greengenes-style lineage consistency
edam_operation: http://edamontology.org/operation_0327
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: QIIME 2
  role: Source of per-feature OTU taxonomy tables whose lineage strings are parsed
    and propagated
  repo: https://docs.qiime2.org/2022.2/data-resources/
- name: Mothur
  role: Source of OTU taxonomy classifications to be rank-parsed; SILVA and Greengenes
    reference files obtained from mothur.org
  repo: https://mothur.org/wiki/silva_reference_files
- name: PathoScope 2
  role: Metagenomic profiler whose feature taxonomies are traced back through the
    taxonomic hierarchy and propagated to missing ranks
- name: Kraken 2
  role: Profiler against SILVA, RefSeq/Standard, and Greengenes indices whose lineage
    labels are parsed rank-by-rank for propagation
  repo: https://benlangmead.github.io/aws-indexes/k2
- name: MetaScope
  role: Package supplying download_refseq() to obtain up-to-date RefSeq reference
    libraries used as propagation inputs
  repo: https://compbiomed.github.io/metascope-docs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:5694c54dcfed0361b6ad08b7c6b6bd559d6ceee1da4be2dbacd24df9dc370c73
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: http://edamontology.org/operation_0327
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# taxonomic-lineage-propagation

## Summary

A post-classification step that fills missing taxonomic ranks (family, genus, species) in OTU/feature metadata by propagating the deepest available lineage label along the reference database's taxonomic path, while annotating each feature with its taxonomic best-hit rank (available granularity).

## When to use

Use when building counts matrices from profiler/reference pairings (QIIME 2, Mothur, PathoScope 2, Kraken 2 vs. SILVA 138, Greengenes 13_8, RefSeq) and feature taxonomy strings contain empty rank fields above the deepest assigned label.

## When NOT to use

- Feature taxonomy metadata is already complete with every rank (kingdom through species) explicitly labeled — propagation would overwrite database-provided labels
- Amplicon pipeline outputs ground-truth species-level calls per feature; here the most-used 16S tools cannot produce high quality genus- or species-level calls, so propagation is only an approximation of granularity
- Reference lineages from a database with no rank-prefix conventions are being merged with Greengenes-prefixed lineages without preserving prefixes — stripping prefixes would break Greengenes-style lineage consistency

## Inputs

- Per-profiler OTU/feature counts matrices (QIIME 2, Mothur, PathoScope 2, Kraken 2 outputs)
- Feature taxonomy lineage strings from SILVA 138, Greengenes 13_8, and RefSeq reference libraries
- Greengenes 13_8 taxonomy file (e.g., gg_13_8_99.taxonomy from the Mothur website)

## Outputs

- taxonomic_best_hits.csv — a single table of propagated lineages plus per-database deepest-rank (best-hit granularity) annotations for every feature
- Uniformly populated taxonomy metadata: all rank fields filled above the deepest available label

## How to apply

Load the per-profiler counts matrices produced by QIIME 2, Mothur, PathoScope 2, and Kraken 2 against SILVA 138, Greengenes 13_8, and RefSeq, then parse each feature's lineage string rank-by-rank to find the deepest rank carrying a non-empty label. Copy that deepest label into every higher-rank field (e.g., an order-only Bacillales hit fills the family, genus, and species fields with o_Bacillales), preserving Greengenes-style rank prefixes (e.g., 'o_', 'g_') where present. For each feature and each database, record the deepest rank reached as the taxonomic best-hit / available-granularity annotation. Assemble propagated lineages and best-hit granularity across all profiler-by-database combinations into a single table and write it to taxonomic_best_hits.csv. The rationale is that PathoScope, QIIME 2, and Mothur allow tracing back the taxonomic hierarchy of a microbe, but references differ in how deep their labels go, so cross-pipeline comparisons at genus and species level require uniform rank filling.

## Related tools

- **QIIME 2** (Source of per-feature OTU taxonomy tables whose lineage strings are parsed and propagated) — https://docs.qiime2.org/2022.2/data-resources/
- **Mothur** (Source of OTU taxonomy classifications to be rank-parsed; SILVA and Greengenes reference files obtained from mothur.org) — https://mothur.org/wiki/silva_reference_files
- **PathoScope 2** (Metagenomic profiler whose feature taxonomies are traced back through the taxonomic hierarchy and propagated to missing ranks)
- **Kraken 2** (Profiler against SILVA, RefSeq/Standard, and Greengenes indices whose lineage labels are parsed rank-by-rank for propagation) — https://benlangmead.github.io/aws-indexes/k2
- **MetaScope** (Package supplying download_refseq() to obtain up-to-date RefSeq reference libraries used as propagation inputs) — https://compbiomed.github.io/metascope-docs

## Evaluation signals

- Every feature row in taxonomic_best_hits.csv has all rank fields populated with no empty values above the deepest available label
- Each row carries an explicit best-hit rank annotation per database (available-granularity column is non-empty for every feature-database pair)
- Propagated labels preserve Greengenes-style rank prefixes (o_, f_, g_, s_) where present in the source lineage string
- A feature with an order-level-only hit (e.g., Bacillales) shows the identical label propagated into family, genus, and species fields
- Row/column counts of taxonomic_best_hits.csv match the union of all profiler-by-database combinations in the original counts matrices

## Limitations

- Propagated higher-rank labels reflect only the lowest available taxonomy — a best hit at order level means genus/species fields carry order-level information, not true genus/species calls
- Commonly used 16S bioinformatics tools cannot produce high-quality genus-level or species-level taxonomic calls, so propagation masks rather than fixes this underlying limitation
- Reference libraries retrieved from public repositories may contain contamination (e.g., E. coli 16S in RefSeq vs. T. flagellates scaffolds) requiring additional filtering before lineage propagation; newer RefSeq versions should be processed to remove contamination before use

## Evidence

- [other] For features missing a taxonomic label at a given level, the lowest available taxonomy was propagated along the database's taxonomic path, recording the available granularity (taxonomic best hit).: [evidence span withheld — anchor status: not_found]
- [discussion] PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe: "PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe"
- [abstract] the most-used bioinformatics tools cannot produce high quality genus-level or species-level taxonomic calls: "the most-used bioinformatics tools cannot produce high quality genus-level or species-level taxonomic calls"
- [readme] Mothur Greengenes indices were obtained from the Mothur website.: "Mothur Greengenes indices were obtained from the Mothur website."
- [discussion] we recommend using older RefSeq libraries until newer versions have been processed to remove contamination: "we recommend using older RefSeq libraries until newer versions have been processed to remove contamination"
