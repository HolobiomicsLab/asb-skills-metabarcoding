---
name: taxonomic-rank-completeness-assessment
description: Use when you hold two or more marker-gene reference databases (e.g.,
  GenBank- vs. BOLD-derived COI fastas) and need to quantify per-rank taxonomic overlap,
  unique contributions, and completeness (missing taxa) to justify which databases
  to combine.
when_to_use_negative:
- Your goal is to evaluate classifier accuracy on query samples (mock community or
  guano reads) rather than to characterize reference database composition itself
- Your input is already a dereplicated, taxonomy-table-formatted set of taxa names
  rather than annotated reference fastas requiring arthropod filtering and rank extraction
- You need versioned, packaged BOLD releases — BOLD discontinued packaged versioned
  releases in 2015, so a directly comparable versioned BOLD snapshot cannot be assumed
tools:
- name: seqkit
  role: grep-filtering reference fastas to arthropod records by taxonomy pattern (e.g.,
    'p:Arthropoda')
- name: VSEARCH
  role: converting COI.udb to fasta (--udb2fasta) and collapsing dereplicated identical
    sequences (--derep_fulllength)
  repo: https://github.com/torognes/vsearch
- name: QIIME1 pick_otus.py
  role: clustering Porter arthropod sequences at 100% identity (--similarity 1.0)
    to dereplicate identical records before LCA consensus
- name: create_consensus_taxonomy.py
  role: building LCA consensus taxonomy strings for dereplicated Porter clusters
- name: R
  role: running figure4_database_comparisons.R to tabulate shared/distinct names per
    rank and render the figure4_upsetPlot.png upset plot
  repo: https://github.com/devonorourke/tidybug
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:ffe4b4b8c987eafc881f33cb68733227ddc103c3a04d42bfd72f40178cf5c386
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - QIIME1 pick_otus.py
  - R
  - VSEARCH
  - create_consensus_taxonomy.py
  - csvtk
  - seqkit
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# taxonomic-rank-completeness-assessment

## Summary

Compares arthropod COI reference databases (GenBank-derived Porter, Palmer amptk, and BOLD-derived tidybug) by extracting Class-through-Species taxonomy strings, tabulating shared vs. unique taxonomic names and missing data per rank, and rendering the overlap as an upset plot to show the resources are complementary rather than redundant.

## When to use

Use when you hold two or more marker-gene reference databases (e.g., GenBank- vs. BOLD-derived COI fastas) and need to quantify per-rank taxonomic overlap, unique contributions, and completeness (missing taxa) to justify which databases to combine.

## When NOT to use

- Your goal is to evaluate classifier accuracy on query samples (mock community or guano reads) rather than to characterize reference database composition itself
- Your input is already a dereplicated, taxonomy-table-formatted set of taxa names rather than annotated reference fastas requiring arthropod filtering and rank extraction
- You need versioned, packaged BOLD releases — BOLD discontinued packaged versioned releases in 2015, so a directly comparable versioned BOLD snapshot cannot be assumed

## Inputs

- Porter Eukaryote CO1 v3.2 training fasta (GenBank-derived, from CO1Classifier GitHub release)
- Palmer amptk COI database (COI.udb, converted to fasta via vsearch --udb2fasta)
- tidybug BOLD-derived dereplicated files (boldCOI.derep.fasta, boldCustom.allArth) from OSF deposit https://osf.io/k3eh6/
- figure4_database_comparisons.R R script

## Outputs

- Per-rank taxa tables (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz) with Class-through-Species strings
- Counts of shared and unique taxonomic names per rank and missing-data counts at each rank
- Upset plot of database overlap (figure4_upsetPlot.png)

## How to apply

Obtain each reference fasta (Porter Eukaryote CO1 v3.2 from the CO1Classifier GitHub release, Palmer amptk COI database converted with vsearch --udb2fasta, and tidybug BOLD dereplicated files boldCOI.derep.fasta and boldCustom.allArth from OSF/GitHub). Restrict each to arthropods: filter Palmer with seqkit grep on 'p:Arthropoda' (1,617,885 to 1,565,831 records) and Porter with grep ';Arthropoda;' (1,280,577 to 883,979 records); dereplicate Porter by clustering identical sequences with pick_otus.py --similarity 1.0, applying the create_consensus_taxonomy.py LCA consensus script, and collapsing with vsearch --derep_fulllength to 515,780 distinct sequences. Extract Class-through-Species taxonomy strings into taxa tables (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz), discarding redundant kingdom/phylum fields, then use the figure4_database_comparisons.R script to count shared and distinct names per rank plus records with missing data at each rank, rendering the result as the Figure 4B upset plot (figure4_upsetPlot.png).

## Related tools

- **seqkit** (grep-filtering reference fastas to arthropod records by taxonomy pattern (e.g., 'p:Arthropoda'))
- **VSEARCH** (converting COI.udb to fasta (--udb2fasta) and collapsing dereplicated identical sequences (--derep_fulllength)) — https://github.com/torognes/vsearch
- **QIIME1 pick_otus.py** (clustering Porter arthropod sequences at 100% identity (--similarity 1.0) to dereplicate identical records before LCA consensus)
- **create_consensus_taxonomy.py** (building LCA consensus taxonomy strings for dereplicated Porter clusters)
- **R** (running figure4_database_comparisons.R to tabulate shared/distinct names per rank and render the figure4_upsetPlot.png upset plot) — https://github.com/devonorourke/tidybug

## Examples

```
seqkit grep -r -p 'Arthropoda' palmer.fasta > palmer.arth.fasta
```

## Evaluation signals

- The arthropod-restricted record counts must match expected reductions: Palmer 1,617,885 to 1,565,831 and Porter 1,280,577 to 883,979
- Porter dereplication must yield 515,780 distinct sequences after pick_otus.py --similarity 1.0, consensus LCA, and vsearch --derep_fulllength
- The upset plot must show 12,788 shared genera with 4,251 genera unique to Porter and 3,885 unique to tidybug
- Unique family fractions must appear as 362 (Porter) and 248 (tidybug), and each database must contribute both shared and unique names at Class-through-Species levels
- Missing-data counts must be reported at every rank from Class to Species, with kingdom/phylum fields discarded from taxa strings

## Limitations

- Comparisons are only valid among databases normalized to arthropod records and dereplicated identically; raw (pre-dereplicated) counts are not comparable across resources
- Databases harbor records with missing taxonomic ranks (sequences lacking Family and/or Order were discarded in the BOLD workflow, and no attempt was made to determine how pervasive missing Phylum/Class info could be)
- BOLD had no packaged versioned releases after 2015, so BOLD-derived snapshots cannot be tied to a stable version, complicating reproducibility

## Evidence

- [results] of 12,788 shared genera, 4,251 unique genera were found only in the Porter dataset and 3,885 only in the tidybug dataset, with similarly large unique family fractions (362 Porter, 248 tidybug): [evidence span withheld — anchor status: not_found]
- [methods] filter Palmer with seqkit grep on 'p:Arthropoda' (1,617,885 to 1,565,831 records) and Porter with grep ';Arthropoda;' (1,280,577 to 883,979 records): [evidence span withheld — anchor status: not_found]
- [methods] all sequences that lacked Family and/or Order information were discarded: "all sequences that lacked Family and/or Order information were discarded"
- [abstract] available reference datasets from GenBank and BOLD for the animal marker gene cytochrome oxidase I (COI) can be complementary: "available reference datasets from GenBank and BOLD for the animal marker gene cytochrome oxidase I (COI) can be complementary"
- [readme] Large database files are available at: https://osf.io/k3eh6/: "Large database files are available at: https://osf.io/k3eh6/"
