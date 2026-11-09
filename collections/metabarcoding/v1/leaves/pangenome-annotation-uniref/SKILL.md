---
name: pangenome-annotation-uniref
description: Use when a pangenome or gene-catalog FASTA of coding sequences (e.g.
  an SGB pangenome .ffn) needs UniRef90/UniRef50 functional annotations before core-gene
  screening or marker extraction.
when_to_use_negative:
- Input is raw contigs or reads rather than called coding sequences — the cds seqtype
  assumes a gene-catalog FASTA of CDSs, not unannotated genomic DNA
- Your DIAMOND databases are the v1.1 files formatted for DIAMOND v0.8 but you run
  DIAMOND v0.9+ — the README notes these are incompatible
- The goal is taxonomic profiling of metagenomes itself (use MetaPhlAn 4) rather than
  functional annotation of a gene catalog
edam_operation: http://edamontology.org/operation_0303
edam_topics:
- http://edamontology.org/topic_0121
- http://edamontology.org/topic_0080
tools:
- name: uniref_annotator
  role: 'Core script: DIAMOND search of CDS FASTA against UniRef90/UniRef50 and enforcement
    of UniRef clustering criteria to annotate pangenome gene catalogs'
  repo: biobakery/uniref_annotator
- name: DIAMOND
  role: Sequence-similarity search engine underlying the annotation; requires v0.8+
    and databases formatted accordingly
- name: MetaPhlAn 4
  role: Downstream pipeline whose SGB pangenome construction and marker-gene extraction
    consume the UniRef-annotated gene catalogs
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:7bc764f27f08a9603ccecae9ba96b03ff9c7a37de431d61a1187a100262be9dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/uniref_annotator
  tools:
  - DIAMOND
  - MetaPhlAn 4
  - uniref_annotator
  edam_operation: http://edamontology.org/operation_0303
  edam_topics:
  - http://edamontology.org/topic_0121
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# pangenome-annotation-uniref

## Summary

Annotates SGB pangenome coding-sequence catalogs against UniRef90/UniRef50 (and, per the MetaPhlAn 4 workflow, Uniclust90) using the biobakery uniref_annotator DIAMOND search, appending UniRef accessions to each FASTA header. This annotation step is a prerequisite for building SGB pangenomes and screening species-unique marker genes in the MetaPhlAn 4 database pipeline.

## When to use

Use when a pangenome or gene-catalog FASTA of coding sequences (e.g. an SGB pangenome .ffn) needs UniRef90/UniRef50 functional annotations before core-gene screening or marker extraction.

## When NOT to use

- Input is raw contigs or reads rather than called coding sequences — the cds seqtype assumes a gene-catalog FASTA of CDSs, not unannotated genomic DNA
- Your DIAMOND databases are the v1.1 files formatted for DIAMOND v0.8 but you run DIAMOND v0.9+ — the README notes these are incompatible
- The goal is taxonomic profiling of metagenomes itself (use MetaPhlAn 4) rather than functional annotation of a gene catalog

## Inputs

- Pangenome coding-sequence FASTA file (e.g. my_pangenome.ffn, CDS nucleotide sequences)
- DIAMOND-formatted UniRef90 database (uniref90.dmnd)
- DIAMOND-formatted UniRef50 database (uniref50.dmnd)
- Optional UniRef90-to-UniRef50 representative mapping TSV (map_uniref90_uniref50.tsv)

## Outputs

- Annotated FASTA file (my_pangenome.ffn.annotated) with UniRef90 and UniRef50 accessions appended to each FASTA header, or UniRef90_unknown for sequences failing the mapping criteria

## How to apply

Run uniref_annotator.py on the pangenome CDS FASTA (e.g. my_pangenome.ffn) with --seqtype cds so coding sequences are translated to protein and searched with a blastp-like DIAMOND search against DIAMOND-formatted UniRef90 (uniref90.dmnd) and UniRef50 (uniref50.dmnd) databases. The script enforces UniRef clustering criteria: a sequence receives a UniRef90 annotation only if it has >=90% alignment identity and >=80% mutual coverage of the cluster centroid. Use --transitive-map with the UniRef90-to-UniRef50 mapping file so that a UniRef90 hit resolves its UniRef50 via lookup rather than independent inference. Output is an annotated FASTA whose headers gain UniRef accessions (e.g. >xx|xxxxx|xxxxxxx|UniRef90_ABC|UniRef50_XYZ), which in the MetaPhlAn 4 pipeline feeds SGB pangenome construction and downstream screen of core genes against all other SGBs to extract ~5.1M species-unique markers.

## Related tools

- **uniref_annotator** (Core script: DIAMOND search of CDS FASTA against UniRef90/UniRef50 and enforcement of UniRef clustering criteria to annotate pangenome gene catalogs) — biobakery/uniref_annotator
- **DIAMOND** (Sequence-similarity search engine underlying the annotation; requires v0.8+ and databases formatted accordingly)
- **MetaPhlAn 4** (Downstream pipeline whose SGB pangenome construction and marker-gene extraction consume the UniRef-annotated gene catalogs) — https://github.com/biobakery/MetaPhlAn

## Examples

```
python uniref_annotator.py my_pangenome.ffn --seqtype cds --uniref90db uniref90.dmnd --uniref50db uniref50.dmnd --diamond-options "--threads 8" --transitive-map map_uniref90_uniref50.tsv
```

## Evaluation signals

- Output FASTA headers follow the expected schema: original ID plus |UniRef90_X|UniRef50_Y, with failed mappings marked as UniRef90_unknown
- Annotated sequences satisfy UniRef criteria: >=90% alignment identity and >=80% mutual coverage of the cluster centroid for UniRef90 assignment
- Runtime sanity check: annotating a ~6K-sequence (~10 MB) pangenome such as S. cerevisiae should take ~20 minutes with 8 cores, or ~20 seconds on pre-computed mappings
- In the full MetaPhlAn 4 workflow, annotated SGB pangenomes support the published endpoint: ~5.1M marker genes over 26,970 profiled SGBs, with an average of 189±34 markers per SGB

## Limitations

- Annotations are approximate: searches are made against UniRef cluster representatives, which are not necessarily the cluster seed (longest) sequences
- The v1.1 DIAMOND databases shipped from the Huttenhower lab are formatted for DIAMOND v0.8 and will not work with DIAMOND v0.9+
- If a CDS does not map to a UniRef90, the --transitive-map option cannot resolve UniRef50 and the header is marked UniRef90_unknown
- In the MetaPhlAn 4 pipeline, environmental, nonhost-associated, and under-studied microbial communities remain enriched for sequences not captured by current uSGBs, limiting annotation coverage

## Evidence

- [readme] The script performs a DIAMOND search of the input against the databases, and then enforces UniRef's clustering criteria on the mapping results to annotate the input sequences.: [evidence span withheld — anchor status: external_doc]
- [readme] UniRef90 annotation thresholds: [evidence span withheld — anchor status: external_doc]
- [methods] SGB pangenome annotation step in the MetaPhlAn 4 workflow: [evidence span withheld — anchor status: not_found]
- [readme] Approximation caveat about cluster representatives: [evidence span withheld — anchor status: external_doc]
- [methods] Marker-gene extraction endpoint that depends on annotated pangenomes: [evidence span withheld — anchor status: not_found]
