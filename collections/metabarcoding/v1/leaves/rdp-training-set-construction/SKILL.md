---
name: rdp-training-set-construction
description: Use when building a CO1 (or similar marker-gene) reference training set
  for the RDP classifier from NCBI/BOLD records — e.g. curating taxon lists, applying
  length/keyword filters, and generating FASTA + taxonomy files for classifier training.
when_to_use_negative:
- Your goal is to classify query sequences with an already-trained classifier — the
  repository ships ready-trained rRNAClassifier.properties files and no further training
  is needed from the user.
- You need a fully representative reference database — only ~22% of extant Insecta
  genera have full-length species-rank CO1 barcodes in GenBank, and CO1 lacks a curated
  community-maintained database like SILVA or UNITE.
- Your queries are short fragments requiring species-rank calls — species-rank assignments
  lack a bootstrap cutoff achieving 99% correct assignments for many fragment lengths.
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: RDP classifier v2.12
  role: Trained with the assembled reference FASTA and taxonomy files to produce genus-rank
    (v1) and species-rank (v2) training sets
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Identifies human contaminant CO1 sequences using a custom database comprised
    of only human CO1 sequences
- name: Perl + BioPerl + Ebot
  role: Queries the NCBI taxonomy database to build the three taxon lists (Arthropoda,
    Chordata, outgroups)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:e9e6b04841358aa1b891234fef63a1ddef2199492a3e36fc098529896dc6e305
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - BioPerl
  - Ebot
  - Perl
  - Perl + BioPerl + Ebot
  - RDP classifier v2.12
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# rdp-training-set-construction

## Summary

Assembles RDP classifier training sets for animal CO1 metabarcode taxonomic assignment by curating taxon lists from NCBI taxonomy, retrieving CO1 reference sequences from GenBank, filtering and contaminant-screening them, and writing reference FASTA plus RDP taxonomy files.

## When to use

Use when building a CO1 (or similar marker-gene) reference training set for the RDP classifier from NCBI/BOLD records — e.g. curating taxon lists, applying length/keyword filters, and generating FASTA + taxonomy files for classifier training.

## When NOT to use

- Your goal is to classify query sequences with an already-trained classifier — the repository ships ready-trained rRNAClassifier.properties files and no further training is needed from the user.
- You need a fully representative reference database — only ~22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank, and CO1 lacks a curated community-maintained database like SILVA or UNITE.
- Your queries are short fragments requiring species-rank calls — species-rank assignments lack a bootstrap cutoff achieving 99% correct assignments for many fragment lengths.

## Inputs

- NCBI taxonomy database (queried via Ebot with ORGN/RANK search terms)
- NCBI nucleotide database records for cox1/coxI/CO1/COI gene sequences
- Custom BLAST database of human CO1 sequences (for contaminant screening)
- Formatted taxon lists (complete binomial species names, Arthropoda / Chordata / outgroups)

## Outputs

- CO1 Eukaryote reference FASTA (v1 training set: 29,998 taxa, 912,253 sequences, ≥500 bp)
- RDP taxonomy files mapping sequence identifiers to full lineages
- Trained RDP classifier files (v1 genus rank, v2 species rank, e.g. CO1v1_trained.tar.gz / CO1v2_trained.tar.gz)

## How to apply

Query the NCBI taxonomy database via Perl/BioPerl with the Ebot script using three searches: "Arthropoda"[ORGN] AND "species"[RANK], "Chordata"[ORGN] AND "species"[RANK], and "cellular organisms"[ORGN] AND "species"[RANK] NOT ("Arthropoda"[ORGN] OR "Chordata"[ORGN]) for outgroups, sampling Arthropoda and Chordata because they are the best-represented eukaryote phyla in GenBank. Format the taxon list keeping only complete binomial species names and excluding entries containing sp., nr., aff., and cf. Query the NCBI nucleotide database with the Entrez term "cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]" plus the formatted lists, adding "BARCODE"[keyword] for outgroup taxa, and retain sequences of at least 500 bp with multiple sequences per species allowed. Screen retrieved sequences for human contaminants by BLAST against a custom database of only human CO1 sequences and retrieve each retained sequence's taxonomic lineage. Combine the Arthropoda, Chordata, and outgroup sets into a single reference FASTA, write RDP-format taxonomy files, and train the RDP classifier (v2.12 in the paper) at the genus rank (v1) and species rank (v2); reconstruct and diff the released training files (e.g. CO1v1_trained.tar.gz) to verify.

## Related tools

- **RDP classifier v2.12** (Trained with the assembled reference FASTA and taxonomy files to produce genus-rank (v1) and species-rank (v2) training sets) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Identifies human contaminant CO1 sequences using a custom database comprised of only human CO1 sequences)
- **Perl + BioPerl + Ebot** (Queries the NCBI taxonomy database to build the three taxon lists (Arthropoda, Chordata, outgroups))

## Evaluation signals

- Reconstructed reference FASTA and taxonomy files match the deposited release (e.g. CO1v1_trained.tar.gz / CO1v2_trained.tar.gz) on sequence identifiers, lineages, and rank structure.
- Reference set totals reproduce expected magnitudes: ~29,998 taxa (all ranks) and ~912,253 sequences for CO1 Eukaryote v1, i.e. 'nearly 1 million' CO1 barcode sequences.
- No retained sequence is a human contaminant per the custom human-CO1 BLAST screen, and all sequences are ≥500 bp.
- Taxon list contains only complete binomial species names with no sp., nr., aff., or cf. entries.
- Leave-one-out / cross-validation of the trained set yields the documented minimum bootstrap cutoffs (e.g. genus rank 70 for 500 bp+ sequences to reach 99% correct assignments).

## Limitations

- CO1 reference coverage is incomplete: only 22% of extant Insecta genera (16,285/72,618) are represented by full-length species-rank CO1 barcodes in GenBank, so false negatives occur when congeneric sequences are missing.
- The method requires RDP training files to be regenerated by the user community as GenBank grows; CO1 lacks a curated community-maintained reference database equivalent to SILVA, Greengenes, or UNITE.
- Later repository versions (v3–v5.1.0) add further filters (no nucleotide ambiguities, BOLD BIN consolidation) and bacterial outgroups; cutoffs differ per release and must be used with the matching version.
- No web portal exists for the RDP-based CO1 classification method; portal availability is deferred to future work.

## Evidence

- [methods] Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa as described below using Perl with Bio Perl modules"
- [methods] Taxon list formatting filter: "A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf."
- [methods] Sequence length filter: "Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available."
- [methods] Human contaminant screen: "Human contaminant sequences were identified using BLAST and a custom database comprised of only human CO1 sequences."
- [results] Taxon sampling rationale: "Taxa for the training set were sampled to emphasize Arthropoda and Chordata since these were the best-represented eukaryote phyla in the Gen Bank nucleotide database."
- [readme] README trained-file usage: "This repository contains training sets that can be used with the Ribosomal Database Project classifier (Wang et al., 2007) to taxonomically assign Eukaryote CO1 mtDNA sequences."
