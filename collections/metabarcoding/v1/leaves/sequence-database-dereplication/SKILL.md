---
name: sequence-database-dereplication
description: Use when comparing the taxonomic composition of two or more reference
  sequence databases whose headers embed taxonomy strings, and redundant identical
  sequences must be collapsed to distinct sequences per rank table before counting
  shared/unique taxa.
when_to_use_negative:
- The input is already a dereplicated feature/sequence table (e.g., tidybug's boldCOI.derep.fasta)
  with no redundant copies to collapse
- Sequences lack taxonomy strings in their headers, so the LCA consensus (create_consensus_taxonomy.py)
  step cannot re-assign a lineage after clustering
- The goal is read-count comparison rather than distinct-sequence/taxonomic-name comparison
  — dereplication changes abundance semantics
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3168
tools:
- name: vsearch
  role: Collapses identical sequences (--derep_fulllength) and converts amptk udb
    to FASTA (--udb2fasta) before dereplication
  repo: https://github.com/torognes/vsearch
- name: QIIME1 pick_otus.py
  role: Clusters identical sequences at --similarity 1.0 to find exact duplicates
    before LCA consensus labeling
- name: create_consensus_taxonomy.py
  role: Re-attaches an LCA consensus taxonomy to each cluster of identical sequences
    after header stripping
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: seqkit
  role: Filters the Palmer database to Arthropoda records ('p:Arthropoda') prior to
    dereplication and tabulation
- name: CO1Classifier reference set
  role: Source of the Porter Eukaryote CO1 training FASTA that is dereplicated and
    compared
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:bf133856cd100555bd51f375f9e04e06a08fb25c37ee9c61eb799efeca35174a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - CO1Classifier reference set
  - QIIME1 pick_otus.py
  - R
  - create_consensus_taxonomy.py
  - csvtk
  - seqkit
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.1
n_claims: 10
---

# sequence-database-dereplication

## Summary

Dereplicate large COI amplicon reference databases (GenBank-derived Porter, amptk Palmer, BOLD-derived tidybug) so that taxonomic composition comparisons count distinct sequences rather than redundant copies, collapsing identical sequences with pick_otus.py at 1.0 similarity and vsearch --derep_fulllength.

## When to use

Use when comparing the taxonomic composition of two or more reference sequence databases whose headers embed taxonomy strings, and redundant identical sequences must be collapsed to distinct sequences per rank table before counting shared/unique taxa.

## When NOT to use

- The input is already a dereplicated feature/sequence table (e.g., tidybug's boldCOI.derep.fasta) with no redundant copies to collapse
- Sequences lack taxonomy strings in their headers, so the LCA consensus (create_consensus_taxonomy.py) step cannot re-assign a lineage after clustering
- The goal is read-count comparison rather than distinct-sequence/taxonomic-name comparison — dereplication changes abundance semantics

## Inputs

- Porter Eukaryote CO1 v3.2 training FASTA from the CO1Classifier GitHub release
- Palmer amptk COI database (COI.udb converted to FASTA with vsearch --udb2fasta)
- tidybug dereplicated BOLD files (boldCOI.derep.fasta, boldCustom.allArth files) from the OSF/GitHub deposit
- Taxonomy strings embedded in FASTA headers (semicolon-delimited lineage fields)

## Outputs

- Dereplicated, taxonomy-labeled FASTA per database (e.g., 515,780 distinct Porter arthropod sequences)
- Per-rank taxa tables (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz) with Class-through-Species names and missing-rank counts
- Figure 4B upset plot (figure4_upsetPlot.png) of shared and unique taxonomic names per database

## How to apply

First restrict each database to the taxon of interest (e.g., filter Palmer with seqkit grep on 'p:Arthropoda', dropping 1,617,885 to 1,565,831 records, and Porter with grep ';Arthropoda;', dropping 1,280,577 to 883,979 records). Then dereplicate: strip the taxonomy from the FASTA headers, cluster identical sequences with QIIME1's pick_otus.py --similarity 1.0, re-attach a consensus taxonomy to each cluster using the create_consensus_taxonomy.py LCA approach (adapted from William Walters' script via the Mike Robeson SILVA-formatting instructions), and finally collapse the labeled sequences with vsearch --derep_fulllength (Porter arthropods collapse to 515,780 distinct sequences). Extract Class-through-Species taxonomy strings into taxa tables (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz), discarding redundant kingdom/phylum fields, then tabulate shared vs. unique names and missing-rank counts with the figure4_database_comparisons.R script and render as an upset plot.

## Related tools

- **vsearch** (Collapses identical sequences (--derep_fulllength) and converts amptk udb to FASTA (--udb2fasta) before dereplication) — https://github.com/torognes/vsearch
- **QIIME1 pick_otus.py** (Clusters identical sequences at --similarity 1.0 to find exact duplicates before LCA consensus labeling)
- **create_consensus_taxonomy.py** (Re-attaches an LCA consensus taxonomy to each cluster of identical sequences after header stripping) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **seqkit** (Filters the Palmer database to Arthropoda records ('p:Arthropoda') prior to dereplication and tabulation)
- **CO1Classifier reference set** (Source of the Porter Eukaryote CO1 training FASTA that is dereplicated and compared) — https://github.com/terrimporter/CO1Classifier

## Examples

```
vsearch --derep_fulllength tmp_nolabels.fasta
```

## Evaluation signals

- Record counts match the reported intermediate values: Porter arthropod filtering 1,280,577 → 883,979; Palmer arthropod filtering 1,617,885 → 1,565,831; Porter dereplication to 515,780 distinct sequences
- Every output header after dereplication carries a Class-through-Species lineage (LCA consensus), with no residual kingdom/phylum duplication in the taxa tables
- The Figure 4B upset plot shows both shared and database-unique taxonomic names at each rank, including 12,788 shared genera, 4,251 Porter-only genera, and 3,885 tidybug-only genera
- Unique family fractions are large and asymmetric (362 Porter-only vs. 248 tidybug-only), consistent with complementarity rather than redundancy
- Sequences lacking Family and/or Order information are counted separately as missing-data records per rank rather than silently dropped

## Limitations

- Database snapshots are time-stamped and versioned inconsistently: BOLD discontinued packaged versioned releases in 2015, and no single curated versioned COI resource is widely used, complicating comparisons among studies
- It is unclear whether the tidybug BOLD-derived database contents match those used by the BOLD Classification API, since the API does not document its database or classification parameters
- No attempt was made to determine how pervasive missing Phylum/Class information is in BOLD records, so completeness estimates at those ranks are under-characterized

## Evidence

- [methods] we can dereplicate the `tmp_nolabels.fasta` directly: "we can dereplicate the `tmp_nolabels.fasta` directly:
``
nvsearch \
--derep_fulllength tmp_nolabels.fasta"
- [methods] We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [readme] Methods and data describing filtering COI datasets and database construction.: "# tidybug
Methods and data describing filtering COI datasets and database construction."
- [readme] This repository contains training sets that can be used with the Ribosomal Database Project classifier (Wang et al., 2007) to taxonomically assign Eukaryote CO1 mtDNA sequences.: [evidence span withheld — anchor status: not_found]
- [methods] incorporated the consensus approach to reclassifying taxa: "incorporated the [consensus approach to reclassifying taxa](https://gist.github.com/walterst/9ddb926fece4b7c0e12c) script written by William (Tony) Walters"
