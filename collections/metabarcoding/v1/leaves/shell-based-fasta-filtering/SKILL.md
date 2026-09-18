---
name: shell-based-fasta-filtering
description: Use when you need to restrict a multi-million-record COI reference FASTA
  (e.g. Palmer amptk COI or Porter GenBank training fasta) to a taxon such as Arthropoda,
  strip taxonomy headers, dereplicate identical sequences, or normalize header/taxonomy
  formats before database comparison.
when_to_use_negative:
- Input is already a dereplicated, taxonomically restricted database (e.g. the tidybug
  boldCOI.derep.fasta), so re-filtering and re-dereplicating would discard taxonomy
  information without changing sequence content
- The goal is taxonomic classification of query sequences against a reference — this
  skill only constructs and filters references, and does not assign taxonomy to samples
- Records are needed at full kingdom/phylum granularity — the taxa extraction deliberately
  discards redundant kingdom/phylum fields and Class-through-Species strings are the
  target
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: seqkit
  role: Regex-based sequence filtering of the Palmer amptk COI FASTA to Arthropoda
    records ('p:Arthropoda')
- name: VSEARCH
  role: Converts COI.udb to FASTA (--udb2fasta) and dereplicates full-length identical
    sequences (--derep_fulllength)
  repo: https://github.com/torognes/vsearch
- name: QIIME1 pick_otus.py
  role: Clusters identical Porter sequences at --similarity 1.0 to identify duplicates
    before LCA consensus
- name: create_consensus_taxonomy.py
  role: Applies LCA consensus taxonomy to the clustered Porter dereplication intermediates
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: csvtk
  role: Shell-level tabular handling of the taxa tables extracted per rank
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:8d33d0bd46d6c8eead2c8f67f89a430681bc4993b6ed79b69e5719ceb0372694
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 9
---

# shell-based-fasta-filtering

## Summary

A command-line skill for subsetting, dereplicating, and reformatting large COI reference FASTA files (GenBank-derived Porter/CO1Classifier, Palmer amptk COI, and BOLD-derived tidybug databases) using seqkit, grep, seqkit/vsearch dereplication, and QIIME pick_otus.py.

## When to use

Use when you need to restrict a multi-million-record COI reference FASTA (e.g. Palmer amptk COI or Porter GenBank training fasta) to a taxon such as Arthropoda, strip taxonomy headers, dereplicate identical sequences, or normalize header/taxonomy formats before database comparison.

## When NOT to use

- Input is already a dereplicated, taxonomically restricted database (e.g. the tidybug boldCOI.derep.fasta), so re-filtering and re-dereplicating would discard taxonomy information without changing sequence content
- The goal is taxonomic classification of query sequences against a reference — this skill only constructs and filters references, and does not assign taxonomy to samples
- Records are needed at full kingdom/phylum granularity — the taxa extraction deliberately discards redundant kingdom/phylum fields and Class-through-Species strings are the target

## Inputs

- Porter Eukaryote CO1 v3.2 training FASTA (CO1Classifier GitHub release)
- Palmer amptk COI reference database (COI.udb, converted to FASTA with vsearch --udb2fasta)
- tidybug dereplicated BOLD files (boldCOI.derep.fasta, boldCustom.allArth files from OSF/GitHub deposit)

## Outputs

- Arthropod-restricted FASTA files (e.g. 1,565,831 Palmer records; 883,979 Porter records)
- Dereplicated Porter arthropod FASTA (515,780 distinct sequences with LCA consensus taxonomy)
- Taxa tables per database (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz) with Class-through-Species strings

## How to apply

Restrict each reference dataset to the target clade in the shell: filter the Palmer amptk COI database with seqkit grep using the pattern 'p:Arthropoda' (reducing 1,617,885 to 1,565,831 records), and filter the Porter Eukaryote CO1 v3.2 training fasta with grep on ';Arthropoda;' (reducing 1,280,577 to 883,979 records). To dereplicate the Porter arthropod records, strip taxonomy from headers, cluster identical sequences with pick_otus.py --similarity 1.0, apply the create_consensus_taxonomy.py LCA consensus script, then collapse with vsearch --derep_fulllength (yielding 515,780 distinct sequences). Earlier pipeline filtering steps in the same project removed gap characters and kept only sequences longer than 100 bp, and screened for non-ATCG characters since BOLD records are not exclusively unambiguous DNA. Extract Class-through-Species taxonomy strings into taxa tables (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz) for downstream comparison with figure4_database_comparisons.R.

## Related tools

- **seqkit** (Regex-based sequence filtering of the Palmer amptk COI FASTA to Arthropoda records ('p:Arthropoda'))
- **VSEARCH** (Converts COI.udb to FASTA (--udb2fasta) and dereplicates full-length identical sequences (--derep_fulllength)) — https://github.com/torognes/vsearch
- **QIIME1 pick_otus.py** (Clusters identical Porter sequences at --similarity 1.0 to identify duplicates before LCA consensus)
- **create_consensus_taxonomy.py** (Applies LCA consensus taxonomy to the clustered Porter dereplication intermediates) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **csvtk** (Shell-level tabular handling of the taxa tables extracted per rank)

## Examples

```
vsearch --derep_fulllength tmp_nolabels.fasta
```

## Evaluation signals

- Record counts after clade filtering must match expectations: Palmer 1,617,885 -> 1,565,831 with seqkit grep 'p:Arthropoda'; Porter 1,280,577 -> 883,979 with grep ';Arthropoda;'
- Porter arthropod dereplication must collapse to exactly 515,780 distinct sequences after pick_otus.py --similarity 1.0, LCA consensus, and vsearch --derep_fulllength
- Extracted taxa tables contain Class-through-Species strings with kingdom/phylum fields removed, and records with missing ranks are counted at each level
- The figure4 upset plot (figure4_upsetPlot.png) must show both shared and unique taxonomic names per database at every rank from Class to Species
- Of 12,788 shared genera, unique-genera counts must be 4,251 (Porter-only) and 3,885 (tidybug-only), with unique families of 362 (Porter) and 248 (tidybug)

## Limitations

- Filtering criteria like discarding sequences lacking Family and/or Order information trade completeness for annotation quality, and the article did not attempt to determine how pervasive missing Phylum/Class information is in BOLD records
- BOLD sequences contain non-ATCG (ambiguous) characters, so screening sequence characters is required before dereplication/clustering steps
- BOLD discontinued packaged versioned releases in 2015, so the BOLD-derived database snapshot's composition depends on the download date (24 February 2019 via the bold R package)

## Evidence

- [methods] Filter1: "We removed any gap character then selected only sequences longer than 100 bp to produce a single fasta file formatted for further processing in Vsearch"
- [methods] Filter2: "The BOLD sequences do not exlcusively contain non-ambiguous DNA characters. To see the distribution of which characters are present:"
- [methods] Filter3: "all sequences that lacked Family and/or Order information were discarded"
- [methods] dereplication: "We'll cluster at 99%, 97%, and 95%; this command produces three output fasta files, one per cluster %id"
- [readme] vsearch README: [evidence span withheld — anchor status: external_doc]
