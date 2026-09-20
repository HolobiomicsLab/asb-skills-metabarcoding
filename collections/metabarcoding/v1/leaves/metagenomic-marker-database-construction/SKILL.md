---
name: metagenomic-marker-database-construction
description: Use when building or reproducing a reference marker-gene database from
  a large genome catalog (isolates plus MAGs), e.g. deriving SGBs and unique markers
  underlying MetaPhlAn 4-style taxonomic profiling.
when_to_use_negative:
- Input is already a marker database or abundance table — this skill constructs the
  reference from genomes, not profiles metagenomes
- The goal is profiling samples with MetaPhlAn rather than rebuilding the SGB/marker
  reference underlying it
- Genomes fail the quality gate (completeness <=50% or contamination >=5% by CheckM)
  and would be excluded from the catalog
- Incorporating viral or eukaryotic microbial sequences — the article notes current
  methods do not extensively incorporate them due to unique genomic architectures
  and QC requirements
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: CheckM
  role: Quality-filters the genome catalog, retaining genomes with completeness >50%
    and contamination <5% before clustering
- name: Mash
  role: Dereplicates the quality-filtered genomes at 5% sequence similarity into species-level
    genome bins (SGBs)
- name: uniref_annotator
  role: Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 using DIAMOND,
    enforcing UniRef clustering criteria (>=90% identity, >=80% mutual centroid coverage)
  repo: biobakery/uniref_annotator
- name: MetaPhlAn 4
  role: Downstream profiler whose SGB marker database this skill reconstructs; defines
    unique marker genes for 26,970 species-level genome bins
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:607638e427f8143665c4db695c41b79f3070032b0d1ebf8a0dfc17982320e01c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/uniref_annotator
  tools:
  - CheckM
  - Mash
  - MetaPhlAn 4
  - uniref_annotator
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# metagenomic-marker-database-construction

## Summary

Reconstructs the MetaPhlAn 4 marker-gene database by quality-filtering an integrated catalog of ~1.01M reference genomes and MAGs, dereplicating them into species-level genome bins (SGBs), and extracting species-unique markers from SGB pangenomes.

## When to use

Use when building or reproducing a reference marker-gene database from a large genome catalog (isolates plus MAGs), e.g. deriving SGBs and unique markers underlying MetaPhlAn 4-style taxonomic profiling.

## When NOT to use

- Input is already a marker database or abundance table — this skill constructs the reference from genomes, not profiles metagenomes
- The goal is profiling samples with MetaPhlAn rather than rebuilding the SGB/marker reference underlying it
- Genomes fail the quality gate (completeness <=50% or contamination >=5% by CheckM) and would be excluded from the catalog
- Incorporating viral or eukaryotic microbial sequences — the article notes current methods do not extensively incorporate them due to unique genomic architectures and QC requirements

## Inputs

- Integrated catalog of prokaryotic reference genomes and metagenome-assembled genomes (~1.01M genomes)
- CheckM completeness/contamination estimates per genome
- Gene catalogs (CDS FASTA, e.g. .ffn pangenome files) per SGB
- DIAMOND-formatted UniRef90 and UniRef50 databases plus UniRef90-to-UniRef50 mapping table

## Outputs

- Dereplicated set of 70,927 SGBs (Mash clusters at 5% identity), including 47,190 uSGBs
- SGB pangenomes with UniRef annotations (FASTA headers extended with UniRef90/UniRef50 IDs)
- Species-unique marker gene database of ~5.1M markers covering 26,970 profiled SGBs, averaging 189±34 markers per SGB

## How to apply

Start from the curated prokaryotic genome catalog and quality-filter it with CheckM, retaining genomes with completeness >50% and contamination <5%, which yields 729,195 genomes (560,084 MAGs and 169,111 reference genomes). Dereplicate the filtered genomes with Mash at 5% sequence similarity into species-level genome bins, recovering 70,927 SGBs of which 47,190 are taxonomically uncharacterized (uSGBs), and retain only uSGBs represented by at least 5 metagenome-assembled genomes. Build SGB pangenomes by annotating gene catalogs against UniRef90 and Uniclust90 using the biobakery uniref_annotator (DIAMOND search with >=90% identity and >=80% mutual coverage of the centroid for UniRef90 annotation). Screen core genes of each SGB against the gene sets of all other SGBs to extract species-unique markers, producing ~5.1M marker genes over 26,970 profiled SGBs. Validate the output against the published reference statistics: 26,970 profiled SGBs, 189±34 markers per SGB on average, and 70,927 total SGBs (47,190 uSGBs).

## Related tools

- **CheckM** (Quality-filters the genome catalog, retaining genomes with completeness >50% and contamination <5% before clustering)
- **Mash** (Dereplicates the quality-filtered genomes at 5% sequence similarity into species-level genome bins (SGBs))
- **uniref_annotator** (Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 using DIAMOND, enforcing UniRef clustering criteria (>=90% identity, >=80% mutual centroid coverage)) — biobakery/uniref_annotator
- **MetaPhlAn 4** (Downstream profiler whose SGB marker database this skill reconstructs; defines unique marker genes for 26,970 species-level genome bins) — https://github.com/biobakery/MetaPhlAn

## Examples

```
python uniref_annotator.py my_pangenome.ffn --seqtype cds --uniref90db uniref90.dmnd --uniref50db uniref50.dmnd --diamond-options "--threads 8" --transitive-map map_uniref90_uniref50.tsv
```

## Evaluation signals

- Filtered genome count matches 729,195 (560,084 MAGs and 169,111 reference genomes) after CheckM filtering (completeness >50%, contamination <5%)
- Mash clustering at 5% identity recovers exactly 70,927 SGBs, of which 47,190 are taxonomically uncharacterized uSGBs
- Marker screening yields ~5.1M species-unique marker genes over 26,970 profiled SGBs, with an average of 189±34 markers per SGB
- Annotated FASTA headers follow the expected schema >xx|xxxxx|xxxxxxx|UniRef90_ABC|UniRef50_XYZ, with failures marked UniRef90_unknown
- uSGB retention filter holds: only uSGBs represented by at least 5 metagenome-assembled genomes are kept

## Limitations

- Environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs
- The current methods do not extensively incorporate viral or eukaryotic microbial sequences, due to their unique genomic architectures and quality control requirements
- uniref_annotator's annotation is approximate: comparisons use UniRef cluster representatives, which are not necessarily the cluster seed (longest) sequences
- The README's bundled DIAMOND databases are formatted for DIAMOND v0.8 and are not compatible with DIAMOND v0.9+

## Evidence

- [abstract] SGB construction workflow: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [readme] uniref_annotator criteria: [evidence span withheld — anchor status: external_doc]
- [readme] uniref_annotator output format: [evidence span withheld — anchor status: external_doc]
- [readme] uniref_annotator DIAMOND requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] MAG-aware platform updates: "We expect to continue addressing these challenges in future versions of the methodology, which will also form the basis for other MAG-aware updates of the bioBakery platform"
