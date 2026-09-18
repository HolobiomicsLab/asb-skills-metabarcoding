---
name: species-level-genome-binning
description: Use when building or reproducing a species-level marker-gene reference
  (e.g. MetaPhlAn 4 SGBs) from a large integrated catalog of prokaryotic reference
  genomes and metagenome-assembled genomes (MAGs).
when_to_use_negative:
- The input is a single genome or a small genome set — Mash clustering at 5% identity
  and the ≥5-MAG-per-uSGB filter are designed for large integrated catalogs
- The goal is taxonomic profiling of shotgun metagenomes with an existing marker database
  rather than constructing the database itself
- Input genomes have not been assessed for completeness/contamination — the CheckM
  filter (>50% completeness, <5% contamination) must precede dereplication
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0610
tools:
- name: CheckM
  role: Quality-filters the genome catalog, retaining genomes with completeness >50%
    and contamination <5%
- name: Mash
  role: Dereplicates quality-filtered genomes and clusters them into SGBs at 5% sequence
    similarity
- name: uniref_annotator
  role: Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 via DIAMOND
    search, enforcing UniRef clustering criteria
  repo: biobakery/uniref_annotator
- name: MetaPhlAn 4
  role: Downstream profiler that consumes the SGB-unique marker database for species-level
    taxonomic profiling
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:ca97a0902b03b82f33eb701e7683d37e61a37d21a09e8636fbe1a04a671f55ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/uniref_annotator
  tools:
  - CheckM
  - Mash
  - MetaPhlAn 4
  - uniref_annotator
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# species-level-genome-binning

## Summary

Reconstructs the MetaPhlAn 4 species-level genome bin (SGB) database by quality-filtering a large catalog of reference genomes and MAGs with CheckM, dereplicating with Mash at 5% identity, and extracting SGB-unique marker genes from SGB pangenomes annotated via UniRef90/Uniclust90.

## When to use

Use when building or reproducing a species-level marker-gene reference (e.g. MetaPhlAn 4 SGBs) from a large integrated catalog of prokaryotic reference genomes and metagenome-assembled genomes (MAGs).

## When NOT to use

- The input is a single genome or a small genome set — Mash clustering at 5% identity and the ≥5-MAG-per-uSGB filter are designed for large integrated catalogs
- The goal is taxonomic profiling of shotgun metagenomes with an existing marker database rather than constructing the database itself
- Input genomes have not been assessed for completeness/contamination — the CheckM filter (>50% completeness, <5% contamination) must precede dereplication

## Inputs

- Integrated catalog of ~1.01 M prokaryotic reference genomes and metagenome-assembled genomes (MAGs)
- Gene catalogs (coding-sequence FASTA, e.g. .ffn) for each SGB pangenome
- DIAMOND-formatted UniRef90 and UniRef50 databases plus a UniRef90-to-UniRef50 mapping file

## Outputs

- Set of 70,927 species-level genome bins (SGBs), including 47,190 taxonomically uncharacterized uSGBs
- ~5.1M SGB-unique marker genes covering 26,970 profiled SGBs (~189±34 markers per SGB)
- UniRef-annotated SGB pangenome FASTA headers (e.g. >xx|xxxxx|xxxxxxx|UniRef90_ABC|UniRef50_XYZ)

## How to apply

Start from a curated catalog of ~1.01 M prokaryotic reference genomes and MAGs and quality-filter with CheckM, retaining genomes with completeness >50% and contamination <5%, which yields 729,195 genomes (560,084 MAGs plus 169,111 reference genomes). Dereplicate the filtered genomes with Mash, clustering at 5% sequence similarity to recover 70,927 species-level genome bins (SGBs), of which 47,190 are taxonomically uncharacterized (uSGBs), and retain only uSGBs represented by at least 5 MAGs. Build an SGB pangenome per bin by annotating its gene catalog against UniRef90 and Uniclust90 using the biobakery uniref_annotator, which enforces UniRef clustering criteria (>=90% identity and >=80% mutual coverage of the centroid for UniRef90). Screen the core genes of each SGB against the gene sets of all other SGBs to extract species-unique markers, producing ~5.1M marker genes profiling 26,970 SGBs. Validate against published reference statistics (26,970 profiled SGBs, ~189±34 markers per SGB, 70,927 total SGBs) before deploying the marker database for MetaPhlAn 4 profiling.

## Related tools

- **CheckM** (Quality-filters the genome catalog, retaining genomes with completeness >50% and contamination <5%)
- **Mash** (Dereplicates quality-filtered genomes and clusters them into SGBs at 5% sequence similarity)
- **uniref_annotator** (Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 via DIAMOND search, enforcing UniRef clustering criteria) — biobakery/uniref_annotator
- **MetaPhlAn 4** (Downstream profiler that consumes the SGB-unique marker database for species-level taxonomic profiling) — https://github.com/biobakery/MetaPhlAn

## Examples

```
python uniref_annotator.py my_pangenome.ffn --seqtype cds --uniref90db uniref90.dmnd --uniref50db uniref50.dmnd --diamond-options "--threads 8"
```

## Evaluation signals

- Genome count after CheckM filtering equals 729,195 (560,084 MAGs + 169,111 reference genomes)
- Mash clustering at 5% identity recovers 70,927 SGBs, of which 47,190 are taxonomically uncharacterized (uSGBs)
- Marker extraction yields ~5.1M unique markers profiling exactly 26,970 SGBs
- Average markers per profiled SGB falls in the published range of 189±34
- Annotated FASTA headers from uniref_annotator follow the pattern >xx|xxxxx|xxxxxxx|UniRef90_ABC|UniRef50_XYZ, with failures marked UniRef90_unknown

## Limitations

- Only uSGBs represented by at least 5 metagenome-assembled genomes are retained, excluding rarer uncharacterized taxa
- uniref_annotator's annotation is approximate: comparisons are made with cluster representatives, which are not necessarily the cluster seed (longest) sequences
- The reference DIAMOND databases from the README are formatted for DIAMOND v0.8 and are incompatible with DIAMOND v0.9+
- Even the extended SGB and marker set under-captures environmental, nonhost-associated, and under-studied microbial communities, and viral/eukaryotic sequences are not extensively incorporated

## Evidence

- [abstract] From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [readme] The script performs a DIAMOND search of the input against the databases, and then enforces UniRef's clustering criteria on the mapping results to annotate the input sequences.: [evidence span withheld — anchor status: external_doc]
- [other] Quality-filter the 1.01M-genome catalog with CheckM, retaining genomes with completeness >50% and contamination <5%, yielding 729,195 genomes.: [evidence span withheld — anchor status: not_found]
- [other] Dereplicate the filtered genomes with Mash and cluster them into species-level genome bins (SGBs) at 5% identity, recovering 70,927 SGBs of which 47,190 are taxonomically uncharacterized (uSGBs).: [evidence span withheld — anchor status: not_found]
