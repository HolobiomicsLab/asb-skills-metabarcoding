---
name: taxonomic-coverage-extrapolation
description: Use when the goal is to measure reference database coverage gaps for
  a taxonomic group (e.g. Insecta genera with full-length CO1 species-rank sequences)
  and project when the database will saturate.
when_to_use_negative:
- Input sequences are already a curated training set and the goal is classifier accuracy
  testing (leave-one-out / cross-validation), not database coverage accounting
- The analysis goal is taxonomic assignment of metabarcode queries (use the RDP classifier
  / BLAST classification workflow instead)
- Growth of the reference database is known to be non-linear or driven by discrete
  projects, making linear extrapolation inappropriate
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0610
tools:
- name: Perl
  role: Scripting language used, with BioPerl modules, to query NCBI taxonomy/nucleotide
    databases and assemble the CO1 reference sets used for coverage counting
- name: BioPerl
  role: BioPerl modules provide NCBI Entrez/taxonomy access for retrieving taxon lineages
    in the coverage workflow
- name: Ebot
  role: Ebot script used to query the NCBI taxonomy database and build the formatted
    taxon list from complete binomial species names
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:441dde0f4654b9208b055d65b9463744ad519c25edf0a17da939c1f7dc09f2ee
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BioPerl
  - Ebot
  - Perl
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# taxonomic-coverage-extrapolation

## Summary

Quantifies how completely a reference sequence database (e.g. GenBank CO1) covers the taxonomic diversity of a clade, by counting rank-level taxa with qualifying sequences across database snapshots, and extrapolates the linear growth rate to estimate time until full coverage.

## When to use

Use when the goal is to measure reference database coverage gaps for a taxonomic group (e.g. Insecta genera with full-length CO1 species-rank sequences) and project when the database will saturate.

## When NOT to use

- Input sequences are already a curated training set and the goal is classifier accuracy testing (leave-one-out / cross-validation), not database coverage accounting
- The analysis goal is taxonomic assignment of metabarcode queries (use the RDP classifier / BLAST classification workflow instead)
- Growth of the reference database is known to be non-linear or driven by discrete projects, making linear extrapolation inappropriate

## Inputs

- NCBI taxonomy database query for the target clade (e.g. class Insecta) retrieved via Perl with BioPerl and the Ebot script
- NCBI nucleotide database records matching CO1 gene Entrez terms combined with a formatted taxon list
- A baseline database snapshot/training set (e.g. Insecta Genbank-Genus, March 2013) and a current snapshot (e.g. CO1 Eukaryote v1, August 2016)
- Total count of extant taxa at the target rank (e.g. 72,618 extant Insecta genera)

## Outputs

- Counts of genera with at least one species-rank, full-length (500 bp+) CO1 sequence per snapshot
- Coverage proportions of extant genera (e.g. 12% and 22%) and derived growth rate
- Estimated years until full genus coverage (~27 years)
- List of unrepresented taxa (e.g. orders Grylloblattodea, Zoraptera, Mantophasmatodea, Dermaptera; families Lepidotrichidae, Lepismatidae, Nicoletiidae) as barcoding targets
- Summary table (e.g. insecta_co1_coverage_report.csv)

## How to apply

Build a formatted taxon list from the NCBI taxonomy database retaining only complete Linnean binomial species names and excluding entries containing sp., nr., aff., and cf. Query the NCBI nucleotide database with the Entrez term 'cox1[gene] OR coxI[gene] OR CO1[gene] OR COI[gene]' combined with the taxon list, retrieving the taxonomic lineage for each record. Apply a minimum 500 bp sequence length filter and count genera having at least one species-rank, full-length CO1 sequence for each snapshot (e.g. March 2013 Insecta Genbank-Genus training set vs. August 2016 / CO1 Eukaryote v1). Compute coverage proportions as fractions of the extant genus total (8,679/72,618 = 12% in 2013 vs. 16,285/72,618 = 22% in 2016), derive the growth rate, and linearly extrapolate to estimate years until all extant genera are represented (~27 years). List taxa still absent as targets for future barcoding.

## Related tools

- **Perl** (Scripting language used, with BioPerl modules, to query NCBI taxonomy/nucleotide databases and assemble the CO1 reference sets used for coverage counting)
- **BioPerl** (BioPerl modules provide NCBI Entrez/taxonomy access for retrieving taxon lineages in the coverage workflow)
- **Ebot** (Ebot script used to query the NCBI taxonomy database and build the formatted taxon list from complete binomial species names)

## Evaluation signals

- Recomputed coverage proportion reproduces the reported 22% of extant Insecta genera (16,285/72,618) represented by full-length species-rank sequences
- Baseline snapshot count matches 8,679 genera (12% of 72,618) from the March 2013 Insecta Genbank-Genus training set
- Linear extrapolation from the 2013-to-2016 growth rate yields approximately 27 years until all 72,618 genera are represented
- Formatted taxon list contains no entries with sp., nr., aff., or cf. designations
- All counted sequences pass the minimum 500 bp length filter

## Limitations

- Persistent coverage gaps remain: Insecta orders Grylloblattodea, Zoraptera, Mantophasmatodea, and Dermaptera, plus families Lepidotrichidae, Lepismatidae, and Nicoletiidae, still lack reference sequences in public databases
- Linear extrapolation assumes the observed growth rate stays constant; database growth may slow as the remaining unrepresented taxa become harder to sample
- GenBank annotation errors and singleton genera can distort counts; summarizing to genus or more inclusive ranks mitigates annotation errors but loses species-level resolution

## Evidence

- [methods] Formatted taxon list from complete binomial names: "A formatted taxon list was created using only taxa with complete binomial species names excluding the names containing sp., nr., aff., and cf."
- [methods] Minimum 500 bp length filter: "Sequences were retained if they were at least 500 bp and multiple sequences per species were retained when available."
- [methods] CO1 Entrez search term for nucleotide query: [evidence span withheld — anchor status: not_found]
- [discussion] 22% Insecta genus coverage figure: "22% of extant Insecta genera (16,285/72,618) are now represented by full length sequences identified to the species rank"
- [abstract] Coverage gaps still exist despite growth: "reference databases are becoming more representative of current species diversity but that gaps still exist"
