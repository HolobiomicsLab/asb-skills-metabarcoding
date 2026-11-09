---
name: genome-quality-filtering-checkm
description: Use when building a marker-gene or pangenome database from a large prokaryotic
  genome catalog (isolates + MAGs) and low-quality assemblies must be excluded before
  dereplication/clustering.
when_to_use_negative:
- Input genomes are already quality-controlled or represent a small curated isolate
  set where CheckM thresholds add no value
- The goal is profiling metagenomes against an existing MetaPhlAn 4 database rather
  than reconstructing the SGB database itself
- Working with viral or eukaryotic genomes, which the paper notes are not extensively
  incorporated 'due to their unique genomic architectures and quality control requirements
  relative to bacterial and archaeal genomes'
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: CheckM
  role: Estimates completeness and contamination for each genome in the catalog; genomes
    with completeness >50% and contamination <5% are retained
- name: Mash
  role: Dereplicates the quality-filtered genomes and clusters them into SGBs at 5%
    sequence similarity
- name: uniref_annotator
  role: Annotates SGB gene catalogs against UniRef90/UniRef50 to build SGB pangenomes
    during marker extraction
  repo: https://github.com/biobakery/uniref_annotator
- name: MetaPhlAn 4
  role: Final profiling tool whose SGB and marker database is produced by this pipeline
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:b995b2afb507aa7186a6e4e1b4ac5d3aef56f602cb062af874b9d9bd77950992
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/uniref_annotator
  tools:
  - CheckM
  - Mash
  - MetaPhlAn 4
  - uniref_annotator
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Genome quality filtering with CheckM

## Summary

Quality-filter an integrated catalog of reference and metagenome-assembled genomes with CheckM before dereplication, retaining only genomes with completeness >50% and contamination <5%. This step underpins the MetaPhlAn 4 SGB database, reducing a 1.01M-genome catalog to 729,195 usable genomes (560,084 MAGs and 169,111 reference genomes).

## When to use

Use when building a marker-gene or pangenome database from a large prokaryotic genome catalog (isolates + MAGs) and low-quality assemblies must be excluded before dereplication/clustering.

## When NOT to use

- Input genomes are already quality-controlled or represent a small curated isolate set where CheckM thresholds add no value
- The goal is profiling metagenomes against an existing MetaPhlAn 4 database rather than reconstructing the SGB database itself
- Working with viral or eukaryotic genomes, which the paper notes are not extensively incorporated 'due to their unique genomic architectures and quality control requirements relative to bacterial and archaeal genomes'

## Inputs

- Integrated catalog of ~1.01M prokaryotic reference genomes and metagenome-assembled genomes (MAGs)
- Per-genome CheckM completeness and contamination estimates
- Optional: UniRef90/UniRef50 DIAMOND databases for downstream pangenome annotation

## Outputs

- Quality-filtered genome set (729,195 genomes passing completeness >50%, contamination <5%)
- Mash-derived SGB clusters at 5% identity (70,927 SGBs, 47,190 uSGBs)
- SGB-specific marker gene set (~5.1M markers across 26,970 profiled SGBs)

## How to apply

Apply CheckM to every genome in the integrated catalog to estimate completeness and contamination. Keep only genomes passing the MetaPhlAn 4 criteria: completeness >50% and contamination <5%; in this study this retained 729,195 of 1.01M genomes (560,084 MAGs, 169,111 reference genomes). Pass the filtered set downstream to Mash dereplication at 5% identity to form 70,927 species-level genome bins (SGBs), of which 47,190 are taxonomically uncharacterized (uSGBs). The rationale is that markers derived from contaminated or fragmentary assemblies would produce spurious species-unique markers (~5.1M markers over 26,970 profiled SGBs in the final database). Retain only uSGBs represented by at least 5 MAGs, then build SGB pangenomes annotated against UniRef90/Uniclust90 with the biobakery uniref_annotator.

## Related tools

- **CheckM** (Estimates completeness and contamination for each genome in the catalog; genomes with completeness >50% and contamination <5% are retained)
- **Mash** (Dereplicates the quality-filtered genomes and clusters them into SGBs at 5% sequence similarity)
- **uniref_annotator** (Annotates SGB gene catalogs against UniRef90/UniRef50 to build SGB pangenomes during marker extraction) — https://github.com/biobakery/uniref_annotator
- **MetaPhlAn 4** (Final profiling tool whose SGB and marker database is produced by this pipeline) — https://github.com/biobakery/MetaPhlAn

## Evaluation signals

- Filtered set size: 729,195 genomes retained from the 1.01M-genome catalog (560,084 MAGs + 169,111 reference genomes)
- Total SGB count after Mash clustering at 5% identity equals 70,927, with 47,190 taxonomically uncharacterized uSGBs
- Final marker database matches published reference statistics: 26,970 profiled SGBs and an average of 189±34 markers per SGB
- Annotated pangenome FASTA headers follow the uniref_annotator schema, e.g. '>xx|xxxxx|xxxxxxx|UniRef90_ABC|UniRef50_XYZ', with failures marked 'UniRef90_unknown'
- uSGBs represented by fewer than 5 MAGs are excluded from the marker set

## Limitations

- Completeness >50% and contamination <5% cutoffs trade sensitivity for specificity; the paper flags 'the tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa' as an unaddressed statistical challenge
- Even the resulting uSGB set misses substantial diversity: 'Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs'
- Viral and eukaryotic microbial genomes are largely excluded, so this filtering pipeline does not cover them
- Databases must be updated regularly; the authors plan 'at least two new MetaPhlAn databases per year' to expand profilable diversity

## Evidence

- [methods] Quality-filter the 1.01M-genome catalog with CheckM, retaining genomes with completeness >50% and contamination <5%, yielding 729,195 genomes.: [evidence span withheld — anchor status: not_found]
- [methods] Dereplicate the filtered genomes with Mash and cluster them into species-level genome bins (SGBs) at 5% identity, recovering 70,927 SGBs.: [evidence span withheld — anchor status: not_found]
- [abstract] From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [readme] to receive a UniRef90 annotation, a sequence must have 1) >=90% alignment identify to and 2) >=80% mutual coverage of the centroid: "Specifically, to receive a UniRef90 annotation, a sequence must have 1) >=90% alignment identify to and
2) >=80% mutual coverage of the centroid."
