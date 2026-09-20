---
name: ncbi-taxonomy-database-querying
description: Use when building a taxonomically structured reference sequence set (e.g.,
  CO1 barcodes) from NCBI, and you need a programmatically generated, rank-filtered
  taxon list to drive Entrez nucleotide queries.
when_to_use_negative:
- Your reference data already comes from a curated database such as BOLD, SILVA, Greengenes,
  or UNITE and does not need NCBI taxonomy mining
- You need sequences shorter than 500 bp or unranked/higher-taxonomy-only queries,
  since the formatted taxon list requires complete binomial species names
- You need a web-portal-style interactive classification service; the article notes
  no web portal is yet available and this is a scripted pipeline
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0084
tools:
- name: Perl
  role: Scripting language used with BioPerl to drive the NCBI taxonomy and nucleotide
    database queries
- name: BioPerl
  role: Perl modules used to programmatically access NCBI databases and parse records
- name: Ebot
  role: Script used to build the formatted taxon list from the NCBI taxonomy database
    query
- name: BLAST
  role: Used to screen retrieved sequences for human contaminants against a custom
    human CO1 database before inclusion
provenance:
  source_task_ids:
  - task_008
  - task_001
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:63c7e28bef3e5e0cdb126aa2a4ccc044cd57eef15f5d9b660b5159800baf0c3f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLAST
  - BioPerl
  - Ebot
  - Perl
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# ncbi-taxonomy-database-querying

## Summary

Query the NCBI taxonomy database via Perl/BioPerl with the Ebot script using structured ORGN/RANK search terms, then format the retrieved taxon list and feed it into NCBI nucleotide Entrez queries to assemble CO1 reference sequence sets.

## When to use

Use when building a taxonomically structured reference sequence set (e.g., CO1 barcodes) from NCBI, and you need a programmatically generated, rank-filtered taxon list to drive Entrez nucleotide queries.

## When NOT to use

- Your reference data already comes from a curated database such as BOLD, SILVA, Greengenes, or UNITE and does not need NCBI taxonomy mining
- You need sequences shorter than 500 bp or unranked/higher-taxonomy-only queries, since the formatted taxon list requires complete binomial species names
- You need a web-portal-style interactive classification service; the article notes no web portal is yet available and this is a scripted pipeline

## Inputs

- Clade name and rank specification for an NCBI taxonomy query (e.g., 'Arthropoda'[ORGN] AND 'species'[RANK])
- Formatted taxon list of complete Linnean binomial species names
- Entrez nucleotide query terms ('cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]'; 'BARCODE'[keyword] for outgroups)

## Outputs

- Formatted taxon list (binomial species names, annotations like sp./nr./aff./cf. removed)
- CO1 nucleotide sequence records (≥500 bp) with associated NCBI taxonomic lineages
- Taxonomy and FASTA files suitable for RDP classifier training

## How to apply

Use Perl with BioPerl modules and the Ebot script to query the NCBI taxonomy database with searches such as 'Arthropoda'[ORGN] AND 'species'[RANK], 'Chordata'[ORGN] AND 'species'[RANK], and 'cellular organisms'[ORGN] AND 'species'[RANK] NOT ('Arthropoda'[ORGN] OR 'Chordata'[ORGN]). Format the returned taxon list keeping only complete binomial species names and excluding entries containing sp., nr., aff., and cf. Combine the formatted list with the Entrez term 'cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]' to query the NCBI nucleotide database, adding 'BARCODE'[keyword] for outgroup taxa to restrict to high-quality records. Retrieve the associated taxonomic lineage for each retained sequence (≥500 bp) so downstream RDP classifier training files can be written.

## Related tools

- **Perl** (Scripting language used with BioPerl to drive the NCBI taxonomy and nucleotide database queries)
- **BioPerl** (Perl modules used to programmatically access NCBI databases and parse records)
- **Ebot** (Script used to build the formatted taxon list from the NCBI taxonomy database query)
- **BLAST** (Used to screen retrieved sequences for human contaminants against a custom human CO1 database before inclusion)

## Examples

```
perl ebot.pl --query '"Arthropoda"[ORGN] AND "species"[RANK]' --out arthropoda_taxa.txt; perl query_ncbi.pl --taxa arthropoda_taxa.txt --entrez '"cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]"' --minlen 500
```

## Evaluation signals

- The formatted taxon list contains only complete Linnean binomial species names and no entries containing sp., nr., aff., or cf.
- Retained nucleotide records are all at least 500 bp and carry a retrieved taxonomic lineage
- Outgroup queries include the 'BARCODE'[keyword] restriction, and GenBank entries with the BARCODE keyword indicate high quality records
- Assembled training set counts match published values (e.g., 29,998 taxa and 912,253 sequences for CO1 Eukaryote v1)
- Reconstructed reference FASTA and taxonomy files match the deposited CO1v1_trained.tar.gz and CO1v2_trained.tar.gz releases in sequence identifiers, lineages, and rank structure

## Limitations

- Taxa with incomplete binomial names (sp., nr., aff., cf.) are excluded, so some taxonomic diversity is omitted from the reference set
- NCBI annotation errors can propagate; the article suggests summarizing taxonomic assignments to the genus or more inclusive ranks to mitigate this
- Reference coverage is incomplete — only 22% of extant Insecta genera (16,285/72,618) were represented by full-length species-rank sequences, and orders like Grylloblattodea, Zoraptera, Mantophasmatodea, and Dermaptera lacked reference sequences

## Evidence

- [methods] Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa as described below using Perl with Bio Perl modules: "Three sets of CO1 reference sequences were assembled: 1) Arthropoda, 2) Chordata, and 3) outgroup taxa as described below using Perl with Bio Perl modules"
- [methods] A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf.: "A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf."
- [methods] The NCBI nucleotide database was queried using the Entrez search term “cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene] AND” the formatted taxon lists: "The NCBI nucleotide database was queried using the Entrez search term “cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene] AND” the formatted taxon lists"
- [methods] Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available.: "Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available."
- [discussion] GenBank entries with the BARCODE keyword indicate high quality records: [evidence span withheld — anchor status: not_found]
