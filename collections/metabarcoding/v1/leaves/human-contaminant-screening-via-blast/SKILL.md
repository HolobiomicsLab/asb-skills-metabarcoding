---
name: human-contaminant-screening-via-blast
description: Use when assembling CO1 (cox1/COI) reference sequences from GenBank for
  classifier training and you need to detect and remove human-origin sequences mislabelled
  as other taxa before formatting taxonomy/FASTA files.
when_to_use_negative:
- Input is a set of metabarcode query reads that need taxonomic assignment rather
  than reference-set curation — use the RDP classifier with the trained CO1 sets instead
- Sequences you are screening come from the v5.0.0+ releases where human sequences
  are deliberately included and screening targets bacterial (not human) contaminants
- The reference data are non-CO1 markers (e.g., 16S rDNA SILVA/Greengenes or fungal
  ITS UNITE sets) for which a human-CO1-specific contaminant database is not appropriate
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: BLAST
  role: Sequence similarity search against a custom database of only human CO1 sequences
    to identify and remove human contaminant reference sequences
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:2156034426618d825c55acbb93d91d08bbd0dffa65e84967f2c484bf6f44ed8c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - BioPerl
  - Ebot
  - Perl
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# human-contaminant-screening-via-blast

## Summary

Screen retrieved CO1 reference sequences for human contaminants by BLAST against a custom database comprising only human CO1 sequences, removing contaminant hits before building RDP classifier training sets. This keeps the CO1 Eukaryote reference FASTA free of mislabelled human sequences.

## When to use

Use when assembling CO1 (cox1/COI) reference sequences from GenBank for classifier training and you need to detect and remove human-origin sequences mislabelled as other taxa before formatting taxonomy/FASTA files.

## When NOT to use

- Input is a set of metabarcode query reads that need taxonomic assignment rather than reference-set curation — use the RDP classifier with the trained CO1 sets instead
- Sequences you are screening come from the v5.0.0+ releases where human sequences are deliberately included and screening targets bacterial (not human) contaminants
- The reference data are non-CO1 markers (e.g., 16S rDNA SILVA/Greengenes or fungal ITS UNITE sets) for which a human-CO1-specific contaminant database is not appropriate

## Inputs

- Retrieved CO1 nucleotide sequences (FASTA) from the NCBI nucleotide database with associated taxon lists
- Custom BLAST database of only human CO1 sequences

## Outputs

- Filtered CO1 reference sequence set with human contaminants removed
- Cleaned sequence set feeding the CO1 Eukaryote reference FASTA and RDP taxonomy files

## How to apply

After retrieving CO1 sequences from the NCBI nucleotide database with the Entrez term 'cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]' combined with formatted taxon lists (and 'BARCODE'[keyword] for outgroups), build a custom BLAST database comprised of only human CO1 sequences. BLAST each retrieved reference sequence against this human-CO1-only database; sequences matching human CO1 are flagged as contaminants and removed from the reference set. Retained sequences and their NCBI taxonomic lineages then feed into the combined Arthropoda, Chordata, and outgroup CO1 Eukaryote reference FASTA used to train the RDP classifier v2.12. Later releases of the same pipeline (v4, v5 in the README) extend the same screening logic to also remove bacterial contaminants.

## Related tools

- **BLAST** (Sequence similarity search against a custom database of only human CO1 sequences to identify and remove human contaminant reference sequences)

## Evaluation signals

- No sequence in the retained CO1 Eukaryote reference FASTA has a significant BLAST hit against the human-CO1-only custom database
- Reconstructed reference FASTA and RDP taxonomy files match the deposited CO1v1_trained.tar.gz / CO1v2_trained.tar.gz releases (sequence identifiers, lineages, rank structure)
- Final training-set totals are consistent with the reported compilation: 29,998 taxa (all ranks) and 912,253 sequences
- Taxonomic composition of the cleaned set emphasizes Arthropoda and Chordata as reported in Table 1 / Table S1

## Limitations

- The original 2018 workflow screens only for human contaminants using a human-CO1 database; bacterial contaminant screening was added only in later repository versions (v4/v5)
- Screening depends on the human CO1 database being complete; divergent or divergently annotated contaminants may escape detection
- Later releases (v5) intentionally include human sequences, so applying this filter to those versions would remove sequences by design

## Evidence

- [methods] Human contaminant sequences were identified using BLAST and a custom database comprised of only human CO1 sequences.: "Human contaminant sequences were identified using BLAST and a custom database comprised of only human CO1 sequences."
- [readme] Sequences were screened to remove human and bacterial contaminants.: "Sequences were screened to remove human and bacterial contaminants."
- [methods] Human contaminant sequence screening filter step in the reference-set assembly workflow: "Human contaminant sequences were identified using BLAST and a custom database comprised of only human CO1 sequences."
