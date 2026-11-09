---
name: taxonomic-lineage-retrieval-and-formatting
description: Use when building a CO1 (or similar marker gene) reference training set
  from NCBI-derived sequences and each retained sequence needs its associated taxonomic
  lineage formatted into RDP taxonomy files.
when_to_use_negative:
- The input is already an RDP-trained classifier (rRNAClassifier.properties) — no
  lineage retrieval or reformatting is needed, sequences can be classified directly
- Sequences have not yet passed the 500 bp length filter and human-contaminant screening
  — lineage formatting of unfiltered sequences would propagate contaminants into the
  training set
- The target is BOLD BIN-based or later releases (v4/v5) that mine both GenBank and
  BOLD with nucleotide-ambiguity filters — a different curation workflow applies
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0085
tools:
- name: Perl + BioPerl (Ebot script)
  role: Queries the NCBI taxonomy database with the published taxon searches and builds
    the formatted taxon list used to retrieve sequences and lineages
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Screens retrieved sequences against a custom human CO1 database before lineages
    are attached, so only non-contaminant sequences get formatted
- name: RDP classifier v2.12
  role: Consumes the formatted FASTA and RDP taxonomy files for classifier training
    at genus (v1) and species (v2) ranks
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:f76c6167af10bb1d57ebac02bed423f5c0aeec4ea83e4c5136912c3c7fabfa08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - BioPerl
  - Ebot
  - Perl
  - Perl + BioPerl (Ebot script)
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0085
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# taxonomic-lineage-retrieval-and-formatting

## Summary

Retrieves the full NCBI taxonomic lineage for every retained CO1 reference sequence and formats taxon lists and RDP-compatible taxonomy files so that a training set can be built and validated against a deposited classifier release.

## When to use

Use when building a CO1 (or similar marker gene) reference training set from NCBI-derived sequences and each retained sequence needs its associated taxonomic lineage formatted into RDP taxonomy files.

## When NOT to use

- The input is already an RDP-trained classifier (rRNAClassifier.properties) — no lineage retrieval or reformatting is needed, sequences can be classified directly
- Sequences have not yet passed the 500 bp length filter and human-contaminant screening — lineage formatting of unfiltered sequences would propagate contaminants into the training set
- The target is BOLD BIN-based or later releases (v4/v5) that mine both GenBank and BOLD with nucleotide-ambiguity filters — a different curation workflow applies

## Inputs

- Retained CO1 nucleotide sequences (>=500 bp, human contaminants removed) with NCBI sequence identifiers
- NCBI taxonomy database query results obtained via Perl/BioPerl Ebot script
- Formatted taxon lists derived from complete binomial species names (excluding sp., nr., aff., cf.)
- Deposited training set release files (e.g. CO1v1_trained.tar.gz, CO1v2_trained.tar.gz) for comparison

## Outputs

- Taxonomic lineage string for each retained reference sequence
- RDP-compatible taxonomy files paired with a reference FASTA for the CO1 Eukaryote v1 (genus rank) and v2 (species rank) training sets

## How to apply

After assembling the Arthropoda, Chordata, and outgroup CO1 sequence sets (screened for human contaminants with BLAST against a custom human CO1 database), query NCBI to retrieve the associated taxonomic lineage for each retained sequence. Earlier in the pipeline, use Perl with BioPerl modules and the Ebot script to query the NCBI taxonomy database with the published searches (e.g. "Arthropoda"[ORGN] AND "species"[RANK]) and create a formatted taxon list keeping only complete binomial species names while excluding names containing sp., nr., aff., and cf. Combine the lineage-annotated FASTA sequences with the taxonomy files, then train the RDP classifier (v2.12 in the paper) at the genus rank (v1) or species rank (v2). Finally, download the deposited training files (e.g. CO1v1_trained.tar.gz from the CO1Classifier GitHub releases) and compare sequence identifiers, lineages, and rank structure against the reconstruction to confirm correctness.

## Related tools

- **Perl + BioPerl (Ebot script)** (Queries the NCBI taxonomy database with the published taxon searches and builds the formatted taxon list used to retrieve sequences and lineages) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Screens retrieved sequences against a custom human CO1 database before lineages are attached, so only non-contaminant sequences get formatted)
- **RDP classifier v2.12** (Consumes the formatted FASTA and RDP taxonomy files for classifier training at genus (v1) and species (v2) ranks) — https://github.com/terrimporter/CO1Classifier

## Examples

```
java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar train -t /path/to/CO1v1_trained/rRNAClassifier.properties -s CO1_Eukaryote_ref.fasta -o taxonomy.rdp
```

## Evaluation signals

- The reconstructed reference FASTA and RDP taxonomy files match the deposited CO1v1_trained.tar.gz / CO1v2_trained.tar.gz in sequence identifiers, lineages, and rank structure
- The reconstructed CO1 Eukaryote v1 training set contains 29,998 taxa (all ranks) and 912,253 sequences
- Formatted taxon lists contain only complete binomial species names with no entries containing sp., nr., aff., or cf.
- Training succeeds directly with the RDP classifier without further user training, confirming taxonomy file formatting is correct

## Limitations

- Taxon sampling emphasizes Arthropoda and Chordata because they are the best-represented eukaryote phyla in GenBank, so coverage of other eukaryote lineages is sparse
- Only species with complete binomial names are retained; annotations with sp., nr., aff., or cf. are excluded, which drops partially identified taxa
- Coverage is incomplete: only 22% of extant Insecta genera are represented by full-length species-rank CO1 barcodes in GenBank, so false negatives occur when congeneric sequences are missing from the database

## Evidence

- [methods] Retrieve taxonomic lineage for each sequence: "The associated taxonomic lineage was retrieved for each sequence."
- [methods] Formatted taxon list filters: "A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf."
- [results] Training sets usable directly: "Training sets can be used directly with the RDP classifier and no further training is needed from the user."
- [readme] Repository release verification path: "The latest release can be downloaded from https://github.com/terrimporter/CO1Classifier/releases .  The trained files ready to be used with the RDP Classifier are available"
- [results] Taxa sampled to emphasize Arthropoda and Chordata: "Taxa for the training set were sampled to emphasize Arthropoda and Chordata since these were the best-represented eukaryote phyla in the Gen Bank nucleotide database."
