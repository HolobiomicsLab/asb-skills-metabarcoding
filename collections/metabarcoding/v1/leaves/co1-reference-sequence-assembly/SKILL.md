---
name: co1-reference-sequence-assembly
description: Use when building a CO1 (cox1/COI) reference training set from NCBI GenBank
  for taxonomic classification of metabarcode reads, e.g. sampling taxa by phylum
  and filtering sequences by length, name completeness, and contamination.
when_to_use_negative:
- Input sequences are shorter than 500 bp or contain nucleotide ambiguities — the
  assembly filters these out, so they do not belong in the reference set
- The goal is classifying non-CO1 markers (e.g. prokaryote 16S rDNA or fungal ITS)
  — curated databases such as SILVA, Greengenes, or UNITE already exist for those
  markers
- A trained CO1 classifier already exists (e.g. the terrimporter/CO1Classifier releases)
  — download and use it rather than rebuilding the training set
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0659
tools:
- name: Ebot
  role: Perl/BioPerl script used to query the NCBI taxonomy database with the three
    phylum-level ORGN/RANK searches to build the taxon list
- name: BLAST
  role: Screens retrieved CO1 sequences for human contaminants against a custom database
    of only human CO1 sequences
- name: RDP classifier
  role: Trained (v2.12) on the assembled reference FASTA and taxonomy files at genus
    rank (v1) and species rank (v2)
  repo: https://github.com/terrimporter/CO1Classifier
- name: CO1Classifier repository releases
  role: Deposited training sets (CO1v1_trained.tar.gz, CO1v2_trained.tar.gz) used
    to verify the reconstructed reference FASTA and taxonomy files match the original
    deposit
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:9e6cd4865e2a6f95401054644c5a9966ed774879a1242ac3769c1f32ecb50b34
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - BioPerl
  - CO1Classifier repository releases
  - Ebot
  - Perl
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# co1-reference-sequence-assembly

## Summary

Assembles a CO1 mitochondrial DNA reference/training set for the RDP classifier by sampling taxa from the NCBI taxonomy database (emphasizing Arthropoda and Chordata), retrieving CO1 sequences from the NCBI nucleotide database, filtering by length and annotation quality, and formatting FASTA plus RDP taxonomy files for training.

## When to use

Use when building a CO1 (cox1/COI) reference training set from NCBI GenBank for taxonomic classification of metabarcode reads, e.g. sampling taxa by phylum and filtering sequences by length, name completeness, and contamination.

## When NOT to use

- Input sequences are shorter than 500 bp or contain nucleotide ambiguities — the assembly filters these out, so they do not belong in the reference set
- The goal is classifying non-CO1 markers (e.g. prokaryote 16S rDNA or fungal ITS) — curated databases such as SILVA, Greengenes, or UNITE already exist for those markers
- A trained CO1 classifier already exists (e.g. the terrimporter/CO1Classifier releases) — download and use it rather than rebuilding the training set

## Inputs

- NCBI taxonomy database query results per phylum-level search (Ebot script)
- Formatted taxon list of complete binomial species names (excluding sp., nr., aff., cf.)
- NCBI nucleotide records matching the CO1 Entrez term (cox1/COI gene), including BARCODE-keyword outgroup records
- Custom BLAST database of only human CO1 sequences

## Outputs

- CO1 Eukaryote reference FASTA (e.g. v1: 912,253 sequences, 29,998 taxa across all ranks)
- RDP taxonomy files (full lineages per sequence) for genus-rank (v1) and species-rank (v2) training
- Trained RDP classifier files (e.g. CO1v1_trained.tar.gz, CO1v2_trained.tar.gz)

## How to apply

Query the NCBI taxonomy database via Perl/BioPerl with the Ebot script using three searches: "Arthropoda"[ORGN] AND "species"[RANK], "Chordata"[ORGN] AND "species"[RANK], and "cellular organisms"[ORGN] AND "species"[RANK] NOT ("Arthropoda"[ORGN] OR "Chordata"[ORGN]) as outgroups. Build a formatted taxon list keeping only complete binomial species names and excluding entries containing sp., nr., aff., and cf.; query the NCBI nucleotide database with the Entrez term "cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]" plus each taxon list, adding "BARCODE"[keyword] for outgroup taxa. Retain sequences of at least 500 bp (multiple sequences per species allowed), screen out human contaminants by BLAST against a custom database of only human CO1 sequences, and retrieve the taxonomic lineage for each retained sequence. Combine the Arthropoda, Chordata, and outgroup sets into a reference FASTA with RDP taxonomy files, then train the RDP classifier (v2.12) at the genus rank (v1) and species rank (v2); taxon sampling emphasizes Arthropoda and Chordata because these are the best-represented eukaryote phyla in GenBank.

## Related tools

- **Ebot** (Perl/BioPerl script used to query the NCBI taxonomy database with the three phylum-level ORGN/RANK searches to build the taxon list)
- **BLAST** (Screens retrieved CO1 sequences for human contaminants against a custom database of only human CO1 sequences)
- **RDP classifier** (Trained (v2.12) on the assembled reference FASTA and taxonomy files at genus rank (v1) and species rank (v2)) — https://github.com/terrimporter/CO1Classifier
- **CO1Classifier repository releases** (Deposited training sets (CO1v1_trained.tar.gz, CO1v2_trained.tar.gz) used to verify the reconstructed reference FASTA and taxonomy files match the original deposit) — https://github.com/terrimporter/CO1Classifier

## Examples

```
wget https://github.com/terrimporter/CO1Classifier/releases/download/RDP-COI-v5.1.0/RDP_COIv5.1.0.zip && unzip RDP_COIv5.1.0.zip
```

## Evaluation signals

- Reconstructed CO1 Eukaryote v1 training set matches the published size: 29,998 taxa (all ranks) and 912,253 sequences
- Sequence identifiers, lineages, and rank structure in the reconstructed FASTA/taxonomy files match the deposited CO1v1_trained.tar.gz and CO1v2_trained.tar.gz from the CO1Classifier releases
- Every retained sequence is at least 500 bp and has an associated taxonomic lineage
- Formatted taxon list contains only complete binomial species names, with no entries containing sp., nr., aff., or cf.
- BLAST against the custom human CO1 database confirms no human contaminant sequences remain in the final reference set

## Limitations

- Coverage is incomplete: only 22% of extant Insecta genera (16,285/72,618) are represented by full-length species-rank CO1 barcodes in GenBank
- CO1 lacks a curated community-maintained reference database equivalent to SILVA, Greengenes, or UNITE, so maintenance falls to the user-community
- False negatives occur when congeneric sequences are missing from the database, of particular concern in quarantine situations; summarizing assignments to genus or more inclusive ranks mitigates database annotation errors

## Evidence

- [methods] Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa"
- [methods] A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf.: "A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf."
- [methods] Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available.: "Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available."
- [results] Taxa for the training set were sampled to emphasize Arthropoda and Chordata since these were the best-represented eukaryote phyla in the Gen Bank nucleotide database.: "Taxa for the training set were sampled to emphasize Arthropoda and Chordata since these were the best-represented eukaryote phyla in the Gen Bank nucleotide database."
- [results] The compiled CO1 Eukaryote v1 training set contains 29,998 taxa (all ranks) and 912,253 sequences.: [evidence span withheld — anchor status: not_found]
- [readme] GenBank sequences were filtered to only include sequences 500bp+, containing no nucleotide ambiguities, and preferrably with a Linnean binomial species name and/or a BOLD BIN: "GenBank sequences were filtered to only include sequences 500bp+, containing no nucleotide ambiguities, and preferrably with a Linnean binomial species name and/or a BOLD BIN"
