---
name: insecta-genus-barcode-gap-analysis
description: Use when the goal is to measure how many extant taxa (e.g. Insecta genera)
  have at least one full-length, species-rank CO1 reference sequence in GenBank, to
  compute coverage growth between database snapshots, or to prioritize unrepresented
  taxa for barcoding.
when_to_use_negative:
- The goal is taxonomic assignment of metabarcode reads rather than reference database
  gap assessment — use the RDP classifier (CO1 Eukaryote v1/v2 training sets) instead
- The marker is not CO1 (e.g. 16S rDNA with SILVA/Greengenes, fungal ITS with UNITE)
  — this gap-analysis workflow is specific to CO1 reference records
- Input sequences are short metabarcode fragments rather than full-length (>=500 bp)
  species-rank reference sequences — the coverage metric requires the 500 bp length
  filter
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: Perl
  role: Scripting language used with BioPerl modules to query the NCBI taxonomy database
    and build the formatted Insecta taxon list
- name: BioPerl
  role: Perl modules used to interface with NCBI taxonomy and process retrieved records
- name: Ebot
  role: Script used together with Perl/BioPerl to build the formatted taxon list from
    NCBI taxonomy queries
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:2f01035ed4861df88d805749414274e97e37d1d678396bbbaa0998fb6531494b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPerl
  - Ebot
  - Perl
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 6
---

# insecta-genus-barcode-gap-analysis

## Summary

Quantifies CO1 reference database coverage at the genus rank for Insecta by mining NCBI taxonomy and nucleotide records, then identifies orders and families (e.g. Grylloblattodea, Zoraptera, Mantophasmatodea, Dermaptera) that still lack species-rank full-length CO1 barcodes and should be targeted for barcoding.

## When to use

Use when the goal is to measure how many extant taxa (e.g. Insecta genera) have at least one full-length, species-rank CO1 reference sequence in GenBank, to compute coverage growth between database snapshots, or to prioritize unrepresented taxa for barcoding.

## When NOT to use

- The goal is taxonomic assignment of metabarcode reads rather than reference database gap assessment — use the RDP classifier (CO1 Eukaryote v1/v2 training sets) instead
- The marker is not CO1 (e.g. 16S rDNA with SILVA/Greengenes, fungal ITS with UNITE) — this gap-analysis workflow is specific to CO1 reference records
- Input sequences are short metabarcode fragments rather than full-length (>=500 bp) species-rank reference sequences — the coverage metric requires the 500 bp length filter

## Inputs

- NCBI taxonomy database query for class Insecta (species-rank records)
- NCBI nucleotide database records matching cox1/coxI/CO1/COI gene Entrez terms
- Taxonomic lineage retrieved for each nucleotide record
- Extant genus count for Insecta (72,618) as denominator
- Prior database snapshot counts (e.g. March 2013 Insecta Genbank-Genus training set: 8,679 genera)

## Outputs

- Coverage proportion per snapshot (genera with >=1 species-rank full-length CO1 sequence / 72,618 extant genera, e.g. 16,285/72,618 = 22%)
- Growth rate between snapshots and extrapolated years to full coverage (~27 years)
- List of unrepresented Insecta orders and families to target for barcoding
- insecta_co1_coverage_report.csv with counts, proportions, and extrapolated years

## How to apply

Build a formatted taxon list from the NCBI taxonomy database (via Perl with BioPerl modules and the Ebot script) retaining only complete Linnean binomial species names and excluding entries containing sp., nr., aff., and cf. Query the NCBI nucleotide database with the Entrez term 'cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]' combined with the Insecta taxon list, and retrieve the taxonomic lineage for each record. Apply a minimum 500 bp sequence length filter and count genera with at least one species-rank, full-length CO1 sequence at each database snapshot (e.g. March 2013 baseline vs. August 2016 / CO1 Eukaryote v1). Compute coverage proportions against the 72,618 extant Insecta genera (8,679 genera in 2013, 16,285 in 2016), derive the growth rate, and linearly extrapolate the years until full coverage (~27 years). Report which orders and families (e.g. Grylloblattodea, Zoraptera, Mantophasmatodea, Dermaptera; Lepidotrichidae, Lepismatidae, Nicoletiidae) remain unrepresented and should be targeted.

## Related tools

- **Perl** (Scripting language used with BioPerl modules to query the NCBI taxonomy database and build the formatted Insecta taxon list)
- **BioPerl** (Perl modules used to interface with NCBI taxonomy and process retrieved records)
- **Ebot** (Script used together with Perl/BioPerl to build the formatted taxon list from NCBI taxonomy queries)

## Examples

```
esearch -db nucleotide -query '"cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]" AND Insecta[ORGN]' | efetch -format docsum > insecta_co1_hits.txt
```

## Evaluation signals

- Recomputed coverage proportion reproduces the reported 22% of extant Insecta genera (16,285/72,618) represented by full-length species-rank sequences
- Baseline snapshot count matches the March 2013 Insecta Genbank-Genus training set figure of 8,679 genera, yielding the 12% -> 22% growth trend
- Every counted genus has at least one species-rank CO1 sequence passing the minimum 500 bp length filter
- Linear extrapolation of the growth rate yields ~27 years until all 72,618 Insecta genera are represented
- Output CSV (insecta_co1_coverage_report.csv) contains counts, proportions, and extrapolated years for both snapshots

## Limitations

- CO1 reference coverage is incomplete: only 22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank, so absence of a genus reflects database gaps as well as sampling
- CO1 lacks a curated community-maintained reference database equivalent to SILVA, Greengenes, or UNITE; the user-community bears responsibility for dataset quality
- False negatives occur when congeneric sequences are missing from the database, of particular concern in quarantine situations
- Counts depend on taxonomic name quality: the taxon list must exclude sp., nr., aff., and cf. entries, and GenBank records may carry outdated names relative to BOLD BIN assignments

## Evidence

- [methods] Ebot taxon list construction: "using Perl with Bio Perl modules and the Ebot script37,38"
- [methods] Entrez CO1 query: "The NCBI nucleotide database was queried using the Entrez search term “cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]"
- [methods] 500 bp retention filter: "Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available."
- [discussion] 22% Insecta genus coverage: "22% of extant Insecta genera (16,285/72,618) are now represented by full length sequences identified to the species rank"
- [readme] Training set filtering conventions: "GenBank sequences were filtered to only include sequences 500bp+, containing no nucleotide ambiguities, and preferrably with a Linnean binomial species name"
