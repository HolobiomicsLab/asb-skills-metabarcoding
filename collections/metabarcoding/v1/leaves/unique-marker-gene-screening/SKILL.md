---
name: unique-marker-gene-screening
description: 'Use when building a marker-gene reference database from a large genome
  catalog (reference genomes + MAGs): you must dereplicate genomes into species-level
  bins and extract each bin''s unique core genes for taxonomic profiling.'
when_to_use_negative:
- The goal is taxonomic profiling of metagenomes against an existing database (run
  MetaPhlAn directly instead of rebuilding markers).
- Genomes have not been quality-filtered — bins built from genomes with completeness
  <=50% or contamination >=5% will contaminate the marker screen.
- Sequences are viral or eukaryotic — the current method does not extensively incorporate
  viral or eukaryotic microbial sequences due to their unique genomic architectures
  and QC requirements.
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_3174
tools:
- name: MetaPhlAn 4
  role: Downstream profiler that consumes the SGB-specific marker database; defines
    unique marker genes for 26,970 species-level genome bins
  repo: https://github.com/biobakery/MetaPhlAn
- name: CheckM
  role: Quality-filters the integrated genome catalog (completeness >50%, contamination
    <5%) before SGB clustering
- name: Mash
  role: Dereplicates quality-filtered genomes into species-level genome bins at 5%
    sequence similarity
- name: uniref_annotator
  role: Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 (and Uniclust90)
    with DIAMOND, enforcing UniRef clustering criteria
  repo: biobakery/uniref_annotator
- name: pyphlan
  role: Supporting SegataLab library for the SGB/pangenome build workflow
  repo: SegataLab/pyphlan
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:eef3d0161fe61481a215aaebe40ee4698a7aa77dbb847f52fb3c8321a3ed71e6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - CheckM
  - Mash
  - MetaPhlAn 4
  - pyphlan
  - uniref_annotator
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# unique-marker-gene-screening

## Summary

Derives species-unique marker genes for MetaPhlAn 4 by quality-filtering an integrated catalog of reference genomes and MAGs with CheckM, dereplicating genomes into species-level genome bins (SGBs) with Mash at 5% identity, building SGB pangenomes annotated against UniRef90/Uniclust90, and screening each SGB's core genes against all other SGBs so only species-unique markers are retained.

## When to use

Use when building a marker-gene reference database from a large genome catalog (reference genomes + MAGs): you must dereplicate genomes into species-level bins and extract each bin's unique core genes for taxonomic profiling.

## When NOT to use

- The goal is taxonomic profiling of metagenomes against an existing database (run MetaPhlAn directly instead of rebuilding markers).
- Genomes have not been quality-filtered — bins built from genomes with completeness <=50% or contamination >=5% will contaminate the marker screen.
- Sequences are viral or eukaryotic — the current method does not extensively incorporate viral or eukaryotic microbial sequences due to their unique genomic architectures and QC requirements.

## Inputs

- Integrated catalog of prokaryotic reference genomes and metagenome-assembled genomes (~1.01M genomes, e.g. in FASTA)
- CheckM quality metrics (completeness and contamination) for each genome
- Per-SGB gene catalogs / pangenome coding sequences (ffn FASTA of CDSs)
- HUMAnN2-formatted UniRef90/UniRef50 DIAMOND databases and UniRef90-to-UniRef50 mapping file

## Outputs

- Species-level genome bins (70,927 SGBs, 47,190 uSGBs) clustered at 5% Mash identity
- Species-unique marker gene database (~5.1M markers) covering 26,970 profiled SGBs, averaging 189±34 markers per SGB
- UniRef90/UniRef50-annotated SGB pangenome FASTA headers (e.g. >xx|xxxxx|xxxxxxx|UniRef90_ABC|UniRef50_XYZ)

## How to apply

First quality-filter the genome catalog (e.g. 1.01M prokaryotic reference genomes and MAGs) with CheckM, retaining genomes with completeness >50% and contamination <5% (this yields ~729,195 genomes). Dereplicate the filtered genomes with Mash at 5% sequence similarity to cluster them into species-level genome bins (70,927 SGBs, of which 47,190 are taxonomically uncharacterized uSGBs), and retain only uSGBs represented by at least 5 MAGs. Build each SGB's pangenome by annotating its gene catalog against UniRef90 and Uniclust90 with biobakery/uniref_annotator, which enforces UniRef's clustering criteria via DIAMOND (>=90% alignment identity and >=80% mutual coverage of the centroid for a UniRef90 annotation). Finally, screen the core genes of each SGB against the gene sets of all other SGBs, keeping only species-unique sequences as markers; the decision point at every screen is strict cross-SGB exclusivity, since any gene shared with another bin is discarded. Validation compares the output against the published reference statistics rather than subjective inspection.

## Related tools

- **MetaPhlAn 4** (Downstream profiler that consumes the SGB-specific marker database; defines unique marker genes for 26,970 species-level genome bins) — https://github.com/biobakery/MetaPhlAn
- **CheckM** (Quality-filters the integrated genome catalog (completeness >50%, contamination <5%) before SGB clustering)
- **Mash** (Dereplicates quality-filtered genomes into species-level genome bins at 5% sequence similarity)
- **uniref_annotator** (Annotates SGB pangenome gene catalogs against UniRef90/UniRef50 (and Uniclust90) with DIAMOND, enforcing UniRef clustering criteria) — biobakery/uniref_annotator
- **pyphlan** (Supporting SegataLab library for the SGB/pangenome build workflow) — SegataLab/pyphlan

## Examples

```
python uniref_annotator.py my_pangenome.ffn --seqtype cds --uniref90db uniref90.dmnd --uniref50db uniref50.dmnd --diamond-options "--threads 8" --transitive-map map_uniref90_uniref50.tsv
```

## Evaluation signals

- Genome count after CheckM filtering matches ~729,195 (560,084 MAGs + 169,111 reference genomes) from the 1.01M-genome catalog.
- Mash clustering at 5% identity recovers 70,927 SGBs, of which 47,190 are taxonomically uncharacterized (uSGBs).
- Final marker database matches published statistics: 26,970 profiled SGBs and ~5.1M unique marker genes, averaging 189±34 markers per SGB.
- UniRef90 annotation correctness: each annotated CDS has >=90% alignment identity and >=80% mutual coverage of the UniRef centroid (checked via uniref_annotator output headers).
- Cross-SGB exclusivity check: no retained marker gene is present in the gene set of any other SGB.

## Limitations

- Environmental, nonhost-associated, and under-studied microbial communities remain enriched for sequences not captured by current uSGBs, so markers derived this way under-cover those habitats.
- The approach does not extensively incorporate viral or eukaryotic microbial sequences due to their unique genomic architectures and quality control requirements.
- Applying quality control to identify real but rare taxa involves a sensitivity/specificity tradeoff that remains an unaddressed downstream statistical challenge.
- The uniref_annotator sample DIAMOND databases are formatted for DIAMOND v0.8 and are incompatible with DIAMOND v0.9+.

## Evidence

- [abstract] From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins"
- [methods] Screen core genes of each SGB against the gene sets of all other SGBs to extract species-unique markers, producing ~5.1M marker genes over 26,970 profiled SGBs.: [evidence span withheld — anchor status: not_found]
- [methods] Build SGB pangenomes by annotating the gene catalogs against UniRef90 and Uniclust90 using the biobakery uniref_annotator.: [evidence span withheld — anchor status: not_found]
- [readme] to receive a UniRef90 annotation, a sequence must have 1) >=90% alignment identify to and 2) >=80% mutual coverage of the centroid.: [evidence span withheld — anchor status: external_doc]
