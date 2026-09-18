---
name: entrez-search-term-construction
description: Use when assembling CO1 (or other gene) reference sequence sets from
  NCBI for a defined taxon list, and the query must capture all gene-name spellings
  and taxonomic rank constraints so that no reference records are missed.
when_to_use_negative:
- Queries against curated databases such as BOLD, SILVA, or UNITE, where Entrez field
  tags like [gene] and [keyword] do not apply
- Retrieval of short metabarcode query reads or existing training sets (e.g. CO1 Eukaryote
  v1/v2 from github.com/terrimporter/CO1Classifier) — these are consumed, not re-mined
  with Entrez terms
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: Perl
  role: Scripting the taxonomy and nucleotide database queries and assembling the
    reference sets
- name: BioPerl
  role: Programmatic access to NCBI databases used alongside the Entrez search terms
- name: Ebot
  role: Script used with Perl/BioPerl to query the NCBI taxonomy database and build
    the formatted taxon list
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:9241592f87b98c98aea1de0797cc8e09b00c92aca2c5e4b4494721fa9b3ca797
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPerl
  - Ebot
  - Perl
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# entrez-search-term-construction

## Summary

Constructs compound NCBI Entrez queries that combine CO1 gene synonyms (cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]) with formatted Linnean taxon lists to retrieve barcode reference sequences from the NCBI nucleotide and taxonomy databases.

## When to use

Use when assembling CO1 (or other gene) reference sequence sets from NCBI for a defined taxon list, and the query must capture all gene-name spellings and taxonomic rank constraints so that no reference records are missed.

## When NOT to use

- Queries against curated databases such as BOLD, SILVA, or UNITE, where Entrez field tags like [gene] and [keyword] do not apply
- Retrieval of short metabarcode query reads or existing training sets (e.g. CO1 Eukaryote v1/v2 from github.com/terrimporter/CO1Classifier) — these are consumed, not re-mined with Entrez terms

## Inputs

- NCBI taxonomy database (queried via Perl with BioPerl modules and the Ebot script)
- Formatted taxon list of complete Linnean binomial species names (sp., nr., aff., cf. excluded)
- Target clade name (e.g. Arthropoda, Insecta) for [ORGN]-qualified taxonomy queries

## Outputs

- CO1 reference sequence records retrieved from the NCBI nucleotide database
- Associated taxonomic lineage for each retrieved sequence

## How to apply

First build a formatted taxon list from the NCBI taxonomy database using rank-qualified terms (e.g. "Arthropoda"[ORGN] AND "species"[RANK]), retaining only complete binomial species names and excluding names containing sp., nr., aff., and cf. Then query the NCBI nucleotide database with the Entrez term combining all four CO1 gene synonyms — "cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]" — AND-ed with the formatted taxon list, so records are matched on gene annotation regardless of spelling variant. For outgroup taxa, append the additional term "BARCODE"[keyword] to restrict to high-quality barcode records. Retrieve the associated taxonomic lineage for each returned sequence so downstream coverage counts and training-set assembly can proceed; later filtering (e.g. minimum 500 bp length) is applied to the retrieved records, not to the Entrez term itself.

## Related tools

- **Perl** (Scripting the taxonomy and nucleotide database queries and assembling the reference sets)
- **BioPerl** (Programmatic access to NCBI databases used alongside the Entrez search terms)
- **Ebot** (Script used with Perl/BioPerl to query the NCBI taxonomy database and build the formatted taxon list)

## Evaluation signals

- Each retrieved record has an associated taxonomic lineage, confirming the AND-combination of the gene and taxon terms worked
- The formatted taxon list contains no names with sp., nr., aff., or cf. tokens
- Outgroup records all carry the BARCODE keyword, since the additional "BARCODE"[keyword] term was appended
- Recovered sequence counts are reproducible when the same Entrez term and taxon list are re-run against a dated GenBank snapshot (e.g. the August 2016 / CO1 Eukaryote v1 snapshot)

## Limitations

- Entrez [gene]-field matching depends on GenBank annotation quality; mis-annotated or missing cox1/COI feature tags will be missed
- Only records with complete binomial species names (no sp., nr., aff., cf.) are usable, which truncates the retrievable reference set
- Results are snapshot-dependent; the paper's Insecta coverage (22% of 72,618 genera in 2016) changes as GenBank grows, so counts must be tied to a dated snapshot

## Evidence

- [methods] The NCBI nucleotide database was queried using the Entrez search term for CO1 gene: "The NCBI nucleotide database was queried using the Entrez search term “cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene] AND” the formatted taxon lists"
- [methods] A formatted taxon list was created using only taxa with complete binomial species names: "A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf."
- [methods] For the outgroup taxa, the additional term BARCODE keyword was used: "For the outgroup taxa, the additional term “BARCODE”[keyword] was used."
- [methods] Sequences were retained if they were at least 500 bp: "Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available."
- [readme] This version includes eukaryote COI sequences mined from GenBank: "This version includes eukaryote COI sequences mined from GenBank [February 2023]. This version includes records deposited between 1982 - 2022 (inclusive)."
