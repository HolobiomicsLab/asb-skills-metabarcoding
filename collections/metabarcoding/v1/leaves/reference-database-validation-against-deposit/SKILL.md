---
name: reference-database-validation-against-deposit
description: Use when a rebuilt CO1 Eukaryote training set (Arthropoda, Chordata,
  outgroups) must be checked against a published GitHub/Zenodo release to confirm
  the reference FASTA and RDP taxonomy files are correctly reconstructed before training
  or reuse.
when_to_use_negative:
- The goal is classifier performance evaluation (leave-one-out testing, cross-validation,
  bootstrap cutoff calibration) rather than verifying the training-set files themselves
- Input is a set of query metabarcoding reads to be taxonomically assigned — use the
  trained classifier directly instead of validating reference files
- The deposit under comparison is a later version (v3/v4/v5) whose construction differs,
  e.g. GenBank records filtered for no nucleotide ambiguities and BOLD BIN consolidation
  with newer Linnean binomials — comparisons across versions will not match
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: RDP classifier v2.12
  role: Version of the classifier the deposited training sets were built for; re-training
    the validated reference files with it should reproduce the trained artifacts in
    the deposit
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Upstream tool used to screen human contaminants during reconstruction; its
    filtering must be reproduced for record counts to match the deposit
- name: Ebot (Perl/BioPerl)
  role: Upstream script that queried the NCBI taxonomy database to produce the taxon
    lists underlying the reconstructed reference set
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:2fe61756ff9a0647a67642983ee8ae7bf24c765d1192bd3c49f9edf9f36b04e9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - BioPerl
  - Ebot
  - Ebot (Perl/BioPerl)
  - Perl
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# reference-database-validation-against-deposit

## Summary

Validates a reconstructed CO1 reference training set (reference FASTA plus RDP taxonomy files) by downloading the authors' deposited release from the terrimporter/CO1Classifier GitHub repository and diffing sequence identifiers, taxonomic lineages, and rank structure against the reconstruction. This closes the reproduction loop for the CO1 Eukaryote v1 (genus-rank) and v2 (species-rank) RDP classifier training sets.

## When to use

Use when a rebuilt CO1 Eukaryote training set (Arthropoda, Chordata, outgroups) must be checked against a published GitHub/Zenodo release to confirm the reference FASTA and RDP taxonomy files are correctly reconstructed before training or reuse.

## When NOT to use

- The goal is classifier performance evaluation (leave-one-out testing, cross-validation, bootstrap cutoff calibration) rather than verifying the training-set files themselves
- Input is a set of query metabarcoding reads to be taxonomically assigned — use the trained classifier directly instead of validating reference files
- The deposit under comparison is a later version (v3/v4/v5) whose construction differs, e.g. GenBank records filtered for no nucleotide ambiguities and BOLD BIN consolidation with newer Linnean binomials — comparisons across versions will not match

## Inputs

- Reconstructed CO1 Eukaryote reference FASTA (Arthropoda + Chordata + outgroup CO1 sequences, >=500 bp, human-contaminant-screened)
- Reconstructed RDP taxonomy files (v1 genus-rank and v2 species-rank lineages)
- Deposited release archives from terrimporter/CO1Classifier (e.g. CO1v1_trained.tar.gz, CO1v2_trained.tar.gz, and matching '-ref' training.tar.gz files)

## Outputs

- Per-record comparison report of sequence identifiers, lineages, and rank structure between reconstruction and deposit
- Confirmation (or discrepancy list) that the reconstructed reference FASTA and taxonomy files match the deposited training set pair

## How to apply

After reconstructing the three CO1 sequence sets (Arthropoda, Chordata, and outgroup taxa) via the NCBI taxonomy/nucleotide queries with the Entrez term 'cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]', combine them into the CO1 Eukaryote reference FASTA and write the RDP taxonomy files. Then download the deposited CO1v1_trained.tar.gz and CO1v2_trained.tar.gz (or the corresponding '-ref' version releases containing the original taxonomy and FASTA files) from https://github.com/terrimporter/CO1Classifier/releases. Decompress the archives and compare, record by record, the sequence identifiers, full taxonomic lineages, and rank structure (e.g. Root;...;Genus for v1, species for v2) between your reconstruction and the deposit. A correct reconstruction should match the deposit's record count (29,998 taxa, 912,253 sequences for v1) and produce identical trained classifier artifacts when re-trained with RDP classifier v2.12. The '-ref' releases in the repository exist precisely so users can inspect 'the original files used for training (a taxonomy file and a FASTA file)' alongside the ready-trained rRNAClassifier.properties bundles.

## Related tools

- **RDP classifier v2.12** (Version of the classifier the deposited training sets were built for; re-training the validated reference files with it should reproduce the trained artifacts in the deposit) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Upstream tool used to screen human contaminants during reconstruction; its filtering must be reproduced for record counts to match the deposit)
- **Ebot (Perl/BioPerl)** (Upstream script that queried the NCBI taxonomy database to produce the taxon lists underlying the reconstructed reference set)

## Examples

```
wget https://github.com/terrimporter/CO1Classifier/releases/download/RDP-COI-v5.1.0/RDP_COIv5.1.0.zip && unzip RDP_COIv5.1.0.zip && diff reconstructed_reference.fasta mydata_trained/reference.fasta
```

## Evaluation signals

- Sequence counts match the published totals for the reconstruction: 29,998 taxa (all ranks) and 912,253 sequences in the CO1 Eukaryote v1 training set
- Every sequence identifier in the reconstructed FASTA is present in the deposited reference FASTA with an identical taxonomic lineage string
- Rank structure of the taxonomy files matches the deposit: v1 trained at the genus rank, v2 trained at the species rank from the same sequences
- The downloaded -ref release's original taxonomy and FASTA files can be diffed record-for-record against the reconstruction with no unexplained differences
- Provenance is traceable: reconstructed records carry BARCODE-keyword outgroup sequences and exclude names containing sp., nr., aff., and cf., as in the deposit

## Limitations

- Later repository releases (v3, v4, v5, v5.1.0) use different construction rules (BOLD data releases, no-nucleotide-ambiguity filters, BIN consolidation), so the v1/v2 validation workflow does not transfer unchanged to those versions
- Bootstrap cutoff values published with one release do not apply to other releases (e.g. v3 cutoffs must be used for v3.1/v3.2), so a validated deposit must be paired with its own documented cutoffs
- Only the original v1/v2 reconstruction is anchored by the three dated 2016 NCBI searches; changes in NCBI taxonomy or content over time can prevent exact reproduction

## Evidence

- [methods] Download deposited training-set releases: "These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier"
- [readme] Original training files are also deposited for reference: "The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file) are available as 'version-ref'"
- [methods] Reconstructed sets must reproduce the three-component composition: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa"
- [methods] RDP classifier version used for training the validated sets: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
