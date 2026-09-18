---
name: genbank-nucleotide-sequence-retrieval
description: Use when building or auditing a CO1 (or other gene) reference set from
  GenBank for a taxon of interest, e.g., to measure which genera or species have full-length
  species-rank barcodes and identify coverage gaps.
when_to_use_negative:
- The query sequences are not intended for taxonomic assignment or reference-set building
  — e.g., you already have a curated training set like CO1 Eukaryote v1/v2 or the
  RDP_COI releases and only need to classify queries via the RDP classifier
- You need a community-curated reference database equivalent to SILVA/Greengenes/UNITE
  for CO1 — none exists, and ad hoc GenBank mining will not substitute for targeted
  barcoding of local biota
- Records lack complete binomial species names or contain sp./nr./aff./cf. designations
  — these should be excluded rather than retrieved, per the paper's taxon-list rules
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0621
tools:
- name: Perl
  role: Scripting language used with BioPerl modules to query NCBI databases and assemble
    CO1 reference sequence sets
- name: BioPerl
  role: BioPerl modules used in Perl scripts to access the NCBI taxonomy and nucleotide
    databases and retrieve taxonomic lineages
- name: Ebot
  role: Ebot script used together with Perl/BioPerl to build formatted taxon lists
    from NCBI taxonomy queries
- name: BLAST
  role: Used to screen retrieved sequences for human contaminants against a custom
    database of human CO1 sequences
- name: CO1Classifier repository
  role: Hosts the assembled CO1 reference/training sets (v1–v5.1.0) produced by this
    retrieval workflow
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:a019578b830c2e1ac441cfc6a6a1d2b50b2bf1e3bded4cf9b048ebcc0f37f192
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - BioPerl
  - CO1Classifier repository
  - Ebot
  - Perl
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# genbank-nucleotide-sequence-retrieval

## Summary

Retrieves and curates GenBank CO1 reference sequences for a taxonomic group by querying the NCBI taxonomy database to build a formatted taxon list, then querying the NCBI nucleotide database with an Entrez CO1 gene term and filtering records by sequence length and taxonomic rank. It underpins construction of CO1 reference sets (e.g., the CO1 Eukaryote v1 training set) and quantification of database coverage gaps.

## When to use

Use when building or auditing a CO1 (or other gene) reference set from GenBank for a taxon of interest, e.g., to measure which genera or species have full-length species-rank barcodes and identify coverage gaps.

## When NOT to use

- The query sequences are not intended for taxonomic assignment or reference-set building — e.g., you already have a curated training set like CO1 Eukaryote v1/v2 or the RDP_COI releases and only need to classify queries via the RDP classifier
- You need a community-curated reference database equivalent to SILVA/Greengenes/UNITE for CO1 — none exists, and ad hoc GenBank mining will not substitute for targeted barcoding of local biota
- Records lack complete binomial species names or contain sp./nr./aff./cf. designations — these should be excluded rather than retrieved, per the paper's taxon-list rules

## Inputs

- NCBI taxonomy database queries for a target taxon (e.g., class Insecta) and rank (e.g., 'species'[RANK])
- Formatted taxon list of complete Linnean binomial species names (excluding sp., nr., aff., cf.)
- NCBI nucleotide database (Entrez) queries with CO1 gene terms: cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]
- Optional BARCODE keyword filter for high-quality outgroup records

## Outputs

- Formatted taxon list file usable to constrain nucleotide database searches
- Retrieved CO1 sequences (FASTA) with associated taxonomic lineage per record
- Filtered reference sequence sets (≥500 bp, no ambiguities, contaminants removed) for classifier training
- Counts/proportions of genera with species-rank full-length CO1 sequences (coverage report, e.g., insecta_co1_coverage_report.csv)

## How to apply

First query the NCBI taxonomy database (e.g., for class Insecta, using search terms such as '"Arthropoda"[ORGN] AND "species"[RANK]') and build a formatted taxon list retaining only complete Linnean binomial species names, excluding entries containing sp., nr., aff., and cf. Then query the NCBI nucleotide database with the Entrez term 'cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]' combined with the formatted taxon list, retrieving the associated taxonomic lineage for each record. Apply a minimum 500 bp sequence length filter (retaining multiple sequences per species when available) and optionally screen out human and bacterial contaminants and ambiguous nucleotides. Count sequences or genera per taxonomic rank across database snapshots to compute coverage proportions (e.g., 16,285/72,618 ≈ 22% of extant Insecta genera) and to assemble training sets such as CO1 Eukaryote v1/v2 for the RDP classifier. Outgroup taxa can be retrieved with the additional term '"BARCODE"[keyword]' to enrich for high-quality records.

## Related tools

- **Perl** (Scripting language used with BioPerl modules to query NCBI databases and assemble CO1 reference sequence sets)
- **BioPerl** (BioPerl modules used in Perl scripts to access the NCBI taxonomy and nucleotide databases and retrieve taxonomic lineages)
- **Ebot** (Ebot script used together with Perl/BioPerl to build formatted taxon lists from NCBI taxonomy queries)
- **BLAST** (Used to screen retrieved sequences for human contaminants against a custom database of human CO1 sequences)
- **CO1Classifier repository** (Hosts the assembled CO1 reference/training sets (v1–v5.1.0) produced by this retrieval workflow) — https://github.com/terrimporter/CO1Classifier

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Recomputed coverage proportions reproduce reported values, e.g., 22% of extant Insecta genera (16,285/72,618) represented by full-length species-rank CO1 sequences
- The 2013 baseline training set (Insecta Genbank-Genus, 8,679 genera) and the August 2016 snapshot (CO1 Eukaryote v1, 16,285 genera) are both recovered when the same queries and ≥500 bp filter are applied
- All retained records have complete Linnean binomial species names and no sp./nr./aff./cf. designations
- Retained sequences are at least 500 bp with no nucleotide ambiguities, and human/bacterial contaminant screening was performed
- Taxonomic lineage is retrievable for every sequence record in the output

## Limitations

- Persistent gaps remain in the CO1 reference database — orders such as Grylloblattodea, Zoraptera, Mantophasmatodea, and Dermaptera, and families such as Lepidotrichidae, Lepismatidae, and Nicoletiidae, still lack reference sequences
- Only ~22% of extant Insecta genera (16,285/72,618) had full-length species-rank CO1 barcodes in GenBank as of the 2016 snapshot; coverage growth extrapolates to ~27 more years before full genus coverage
- GenBank annotation quality varies; summarizing assignments to genus or more inclusive ranks is recommended to mitigate database annotation errors
- Sequences must be screened for human and bacterial contaminants; v4/v5 pipelines also required <3 ambiguous bases and BOLD BIN consolidation for some releases

## Evidence

- [methods] A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf.: "A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf."
- [methods] NCBI nucleotide database queried with CO1 Entrez terms: [evidence span withheld — anchor status: not_found]
- [methods] 500 bp sequence length filter: "Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available."
- [methods] Perl with BioPerl and Ebot used to assemble CO1 reference sets: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa as described below using Perl with Bio Perl modules"
- [methods] BARCODE keyword filter for high-quality outgroup records: [evidence span withheld — anchor status: not_found]
- [readme] Coverage gap finding from README filtering criteria for newer releases: "GenBank sequences were filtered to only include sequences 500bp+, containing no nucleotide ambiguities, and preferrably with a Linnean binomial species name"
