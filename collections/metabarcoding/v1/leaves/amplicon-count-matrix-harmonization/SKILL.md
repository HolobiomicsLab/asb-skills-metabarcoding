---
name: amplicon-count-matrix-harmonization
description: Use when building a counts matrix from profiler/reference pairings and
  reference lineages leave ranks empty (family, genus, species), so taxonomic labels
  must be propagated to a uniform full-lineage schema.
when_to_use_negative:
- The input reference database already provides explicit, non-empty labels at all
  ranks (kingdom through species), so no propagation is needed.
- The input is already a harmonized feature table with complete lineage annotations
  and best-hit rank metadata.
- Assigning new taxonomy de novo by sequence similarity (e.g., retraining the QIIME
  2 or Mothur naive Bayes classifiers) rather than reconciling existing profiler lineage
  outputs.
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: QIIME 2
  role: Profiler whose OTU counts matrices and traceable taxonomic hierarchies supply
    lineage strings for propagation; SILVA and Greengenes 13_8 indices obtained from
    the QIIME 2 website.
- name: Mothur
  role: Profiler whose counts matrices' lineages are parsed rank-by-rank; SILVA and
    Greengenes 13_8 (Gg_13_8_99) taxonomy files obtained from the Mothur website.
- name: PathoScope 2
  role: Whole-genome profiler whose feature lineages are propagated; SILVA, Greengenes,
    and RefSeq indices were custom-configured for it.
- name: Kraken 2
  role: Profiler run against SILVA (Index Zone), Kraken 2 Standard/RefSeq, and Greengenes
    indices built with kraken2-build --special greengenes; its outputs feed the lineage
    propagation step.
- name: MetaScope
  role: Package whose download_refseq() function supplies up-to-date RefSeq indices
    used to build the profiler/reference pairings being harmonized.
  repo: https://compbiomed.github.io/metascope-docs/index.html
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:293b58fa4bea5b037fbac02d3ce5f13d9a9d1f356f0035e8d43916054b37d71c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Kraken 2
  - MetaScope
  - Mothur
  - PathoScope 2
  - QIIME 2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# amplicon-count-matrix-harmonization

## Summary

Harmonizes OTU/feature taxonomy metadata across per-profiler 16S count matrices (QIIME 2, Mothur, PathoScope 2, Kraken 2 vs SILVA 138, Greengenes 13_8, RefSeq) by propagating the lowest available taxonomic label into higher-rank fields and annotating the taxonomic best-hit granularity per feature and database.

## When to use

Use when building a counts matrix from profiler/reference pairings and reference lineages leave ranks empty (family, genus, species), so taxonomic labels must be propagated to a uniform full-lineage schema.

## When NOT to use

- The input reference database already provides explicit, non-empty labels at all ranks (kingdom through species), so no propagation is needed.
- The input is already a harmonized feature table with complete lineage annotations and best-hit rank metadata.
- Assigning new taxonomy de novo by sequence similarity (e.g., retraining the QIIME 2 or Mothur naive Bayes classifiers) rather than reconciling existing profiler lineage outputs.

## Inputs

- Per-profiler OTU/feature counts matrices from QIIME 2, Mothur, PathoScope 2, and Kraken 2
- Reference-library lineage strings/taxonomy files from SILVA 138, Greengenes 13_8, and RefSeq

## Outputs

- Single harmonized table of propagated full lineages with per-feature, per-database taxonomic best-hit rank annotations (taxonomic_best_hits.csv)
- Counts matrix features assigned uniform taxonomy metadata at every rank

## How to apply

Load the per-profiler counts matrices produced by QIIME 2, Mothur, PathoScope 2, and Kraken 2 against SILVA 138, Greengenes 13_8, and RefSeq, then parse each feature's lineage string rank-by-rank to locate the deepest rank carrying a non-empty label. Copy that deepest label into every higher-rank field (e.g., an order-only Bacillales hit fills family, genus, and species with o_Bacillales), preserving Greengenes-style rank prefixes (o_, f_, g_, s_) where present. This exploits the fact that PathoScope, QIIME 2, and Mothur all allow tracing back the taxonomic hierarchy of a given microbe, enabling upsampling across taxon levels. For each feature and each database, record the deepest rank reached as the taxonomic best hit / available-granularity annotation. Assemble propagated lineages and best-hit granularity across all profiler-by-database combinations into a single table and write it to taxonomic_best_hits.csv.

## Related tools

- **QIIME 2** (Profiler whose OTU counts matrices and traceable taxonomic hierarchies supply lineage strings for propagation; SILVA and Greengenes 13_8 indices obtained from the QIIME 2 website.)
- **Mothur** (Profiler whose counts matrices' lineages are parsed rank-by-rank; SILVA and Greengenes 13_8 (Gg_13_8_99) taxonomy files obtained from the Mothur website.)
- **PathoScope 2** (Whole-genome profiler whose feature lineages are propagated; SILVA, Greengenes, and RefSeq indices were custom-configured for it.)
- **Kraken 2** (Profiler run against SILVA (Index Zone), Kraken 2 Standard/RefSeq, and Greengenes indices built with kraken2-build --special greengenes; its outputs feed the lineage propagation step.)
- **MetaScope** (Package whose download_refseq() function supplies up-to-date RefSeq indices used to build the profiler/reference pairings being harmonized.) — https://compbiomed.github.io/metascope-docs/index.html

## Examples

```
kraken2-build --db ${DBNAME} --special greengenes --threads 1
```

## Evaluation signals

- Every feature row has all rank fields populated with no empty values above the deepest available label.
- Each row carries an explicit taxonomic best-hit rank annotation per database.
- Greengenes-style rank prefixes (o_, f_, g_, s_) are preserved in the propagated lineage fields where present.
- The output assembles propagated lineages consistently across all profiler-by-database combinations and is written to taxonomic_best_hits.csv.
- Spot-check examples: an order-only Bacillales hit must fill family, genus, and species fields with o_Bacillales.

## Limitations

- Reference library caveats: Greengenes 13_8 dates to August 2013, and newer RefSeq versions may require additional filtering and correction to remove contamination before use.
- Propagated labels reflect available granularity, not true assignments — tools cannot reliably make high-quality genus- or species-level calls, so higher-rank fills (e.g., species = o_Bacillales) are placeholders.
- Mock communities provide only a semblance of ground truth, so propagated annotations cannot be validated against perfect expected lineages.

## Evidence

- [discussion] PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe: "PathoScope, QIIME 2, and Mothur all allow the tracing back of the taxonomic hierarchy of a given microbe"
- [readme] QIIME 2 Greengenes indices were obtained from the QIIME 2 website: "QIIME 2 Greengenes indices were obtained from the [QIIME 2 website.](https://docs.qiime2.org/2022.2/data-resources/?highlight=greengenes)"
- [readme] Up-to-date RefSeq indices may be obtained by using the download_refseq() function from the MetaScope package: "Up-to-date RefSeq indices may be obtained by using the `download_refseq()` function from the [MetaScope package]"
- [discussion] this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository: "this is an ongoing problem that demands additional filtering and correction efforts after directly retrieving libraries from the public repository"
