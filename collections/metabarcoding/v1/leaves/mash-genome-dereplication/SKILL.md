---
name: mash-genome-dereplication
description: Use when a genome catalog of hundreds of thousands of reference genomes
  and/or MAGs must be reduced to species-level clusters (SGBs) prior to pangenome
  construction and species-unique marker extraction.
when_to_use_negative:
- Input genomes have not passed quality control (completeness >50%, contamination
  <5%) — clustering unfiltered genomes will inflate SGB counts with fragmented or
  contaminated MAGs
- Goal is below-species (strain-level) clustering — 5% identity is a species-level
  cut point, not a strain dereplication threshold
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3174
tools:
- name: Mash
  role: MinHash-based genome sketching and clustering of the quality-filtered genome
    catalog into SGBs at 5% identity
- name: CheckM
  role: Pre-cluster quality filtering of MAGs and reference genomes by completeness
    and contamination
- name: MetaPhlAn 4
  role: Downstream consumer of the SGB/marker database built from the dereplicated
    bins
  repo: https://github.com/biobakery/MetaPhlAn
- name: uniref_annotator
  role: Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 after dereplication
  repo: https://github.com/biobakery/uniref_annotator
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:2bab8c0564d25917a373bdad407cd8322171e46e0e4f2adeeb83332c03df3874
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - CheckM
  - Mash
  - MetaPhlAn 4
  - uniref_annotator
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mash-genome-dereplication

## Summary

Dereplicates large catalogs of prokaryotic reference genomes and MAGs into species-level genome bins (SGBs) using Mash MinHash sketches at a 5% sequence-similarity threshold, the clustering step underlying the MetaPhlAn 4 marker database.

## When to use

Use when a genome catalog of hundreds of thousands of reference genomes and/or MAGs must be reduced to species-level clusters (SGBs) prior to pangenome construction and species-unique marker extraction.

## When NOT to use

- Input genomes have not passed quality control (completeness >50%, contamination <5%) — clustering unfiltered genomes will inflate SGB counts with fragmented or contaminated MAGs
- Goal is below-species (strain-level) clustering — 5% identity is a species-level cut point, not a strain dereplication threshold

## Inputs

- Integrated catalog of prokaryotic reference and metagenome-assembled genomes (1.01M genomes)
- CheckM completeness/contamination estimates for each genome
- Genome sequences in FASTA format for Mash sketching

## Outputs

- Species-level genome bins (SGBs) clustered at 5% Mash sequence similarity (70,927 SGBs)
- SGB-to-genome membership assignments
- Filtered genome set (729,195 genomes passing CheckM criteria)

## How to apply

First quality-filter the integrated genome catalog with CheckM, retaining only genomes with completeness >50% and contamination <5% (this leaves 729,195 genomes from the MetaPhlAn 4 1.01M-genome catalog: 560,084 MAGs and 169,111 reference genomes). Next sketch and compare all filtered genome pairs with Mash and cluster them into SGBs at 5% sequence similarity, recovering 70,927 SGBs (47,190 taxonomically uncharacterized uSGBs). Retain only uSGBs represented by at least 5 metagenome-assembled genomes. The dereplicated SGBs are then passed to pangenome construction (gene annotation against UniRef90/Uniclust90 via biobakery uniref_annotator) and cross-SGB core-gene screening to extract species-unique markers (~5.1M marker genes over 26,970 profiled SGBs).

## Related tools

- **Mash** (MinHash-based genome sketching and clustering of the quality-filtered genome catalog into SGBs at 5% identity)
- **CheckM** (Pre-cluster quality filtering of MAGs and reference genomes by completeness and contamination)
- **MetaPhlAn 4** (Downstream consumer of the SGB/marker database built from the dereplicated bins) — https://github.com/biobakery/MetaPhlAn
- **uniref_annotator** (Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 after dereplication) — https://github.com/biobakery/uniref_annotator

## Evaluation signals

- Filtered genome count matches ~729,195 genomes (560,084 MAGs + 169,111 reference genomes) when starting from the 1.01M-genome catalog
- Total SGB count matches 70,927, of which 47,190 are taxonomically uncharacterized (uSGBs)
- Number of profiled SGBs with extracted markers matches 26,970, with an average of 189±34 species-unique markers per SGB
- Total marker-gene count is on the order of ~5.1M over the profiled SGBs
- Every uSGB retained in the final set is represented by at least 5 MAGs

## Limitations

- Environmental, nonhost-associated, and under-studied microbial communities are still highly enriched for sequences not captured by current uSGBs
- Clustering granularity is fixed at 5% sequence similarity (species level); it does not resolve strain-level diversity on its own
- Dereplication quality depends on upstream CheckM filtering; genomes failing completeness/contamination thresholds are excluded

## Evidence

- [methods] Quality-filter the 1.01M-genome catalog with CheckM, retaining genomes with completeness >50% and contamination <5%: [evidence span withheld — anchor status: not_found]
- [methods] Dereplicate the filtered genomes with Mash and cluster them into species-level genome bins (SGBs) at 5% identity: [evidence span withheld — anchor status: not_found]
- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [abstract] MetaPhlAn 4, which integrates information from metagenome assemblies and microbial isolate genomes: "Here we present MetaPhlAn 4, which integrates information from metagenome assemblies and microbial isolate genomes for more comprehensive metagenomic taxonomic profiling."
