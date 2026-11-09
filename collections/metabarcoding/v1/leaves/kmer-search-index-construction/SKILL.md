---
name: kmer-search-index-construction
description: Use when rRNA reads (FASTA/FASTQ, amplicon or shotgun) need taxonomy
  or OTU assignment against a full-length rRNA reference, or when a custom reference
  database must be indexed for MAPseq classification.
when_to_use_negative:
- Input is already a MAPseq .mseq output file — use 'mapseq -otucounts' or 'mapseq
  -otutable' for summarization instead of re-indexing
- The bundled default reference dataset suffices for the analysis — no custom indexing
  is needed beyond the standard 'mapseq rawseqs.fa' call
- Sequences are not ribosomal RNA reads and lack an rRNA-appropriate reference; MAPseq's
  confidence cutoffs and OTU clusterings are optimized for rRNA
edam_operation: http://edamontology.org/operation_0338
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: MAPseq
  role: k-mer search engine that indexes the rRNA reference and classifies reads with
    confidence estimates in a single run
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: generates the 'configure' script used to prepare the MAPseq build before index
    construction
  repo: https://github.com/jfmrod/mapseq
- name: GNU make
  role: compiles and installs MAPseq 1.2.6 so the binary can build its search index
    and data files
  repo: https://github.com/jfmrod/mapseq
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:3b0ecc6437a5c3aee2d7dd55efdba9f92e32935de9ead3eb1f3990c0ca6f2f84
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  edam_operation: http://edamontology.org/operation_0338
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# kmer-search-index-construction

## Summary

Construct and use MAPseq's k-mer search index over a full-length rRNA reference database (with its taxonomy/OTU files) to classify amplicon or shotgun rRNA reads with per-level confidence estimates, in a single run.

## When to use

Use when rRNA reads (FASTA/FASTQ, amplicon or shotgun) need taxonomy or OTU assignment against a full-length rRNA reference, or when a custom reference database must be indexed for MAPseq classification.

## When NOT to use

- Input is already a MAPseq .mseq output file — use 'mapseq -otucounts' or 'mapseq -otutable' for summarization instead of re-indexing
- The bundled default reference dataset suffices for the analysis — no custom indexing is needed beyond the standard 'mapseq rawseqs.fa' call
- Sequences are not ribosomal RNA reads and lack an rRNA-appropriate reference; MAPseq's confidence cutoffs and OTU clusterings are optimized for rRNA

## Inputs

- Demultiplexed, quality-filtered FASTA of raw rRNA sequence reads (e.g. rawseqs.fa)
- Full-length rRNA reference database in nucleotide FASTA format (customref.fasta)
- One or more tab-separated taxonomy assignment files (customref.tax) with a '#cutoff:' header declaring identity cutoffs per level
- Optionally a known-identity read benchmark set with ground-truth taxonomic assignments for accuracy evaluation

## Outputs

- MAPseq k-mer search index built over the reference database (data files stored relative to the mapseq binary path)
- .mseq classification file (e.g. rawseqs.fa.mseq) with alignment fields and per-taxonomic-level assignments plus confidence values
- Optional '-otucounts' per-sample summary table or '-otutable' TSV of OTU/taxa counts across multiple samples

## How to apply

Build MAPseq 1.2.6 from source via './configure && make && make install' (run './setup.sh' and './bootstrap' after cloning, requiring autoconf, GNU make, libtool, and libncurses5-dev). Supply a nucleotide FASTA reference of full-length ribosomal RNA sequences plus one or more tab-separated taxonomy files whose '#cutoff:' header line declares per-level identity cutoffs (e.g. '#cutoff: 0.00:0.08 0.70:0.35 ...' for the NCBI taxonomy). MAPseq internally builds its k-mer search index over the reference; the index and data files are searched relative to the binary path, so keep the directory intact or install with './configure --prefix=$HOME/usr'. Classify reads in one run (mapseq rawseqs.fa <ref.fasta> <ref.tax> ...) and interpret the .mseq output, which reports best-hit alignment fields (score, pairwise identity, matches/mismatches/gaps, coordinates) followed by taxonomy labels with combined confidence at each level. Judge correctness by checking that each read line carries a reference id, alignment statistics, and per-level confidences, and by benchmarking F-half accuracy against ground-truth assignments at a combined confidence cutoff of 0.4-0.5.

## Related tools

- **MAPseq** (k-mer search engine that indexes the rRNA reference and classifies reads with confidence estimates in a single run) — https://github.com/jfmrod/mapseq
- **autoconf** (generates the 'configure' script used to prepare the MAPseq build before index construction) — https://github.com/jfmrod/mapseq
- **GNU make** (compiles and installs MAPseq 1.2.6 so the binary can build its search index and data files) — https://github.com/jfmrod/mapseq

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- Each .mseq line contains a query id, reference id (highest alignment score), alignment score, pairwise identity, matches/mismatches/gaps, and coordinates, followed by taxonomy labels with combined confidence values
- At a combined confidence cutoff of 0.4-0.5, the benchmark F½ score against ground-truth assignments matches or exceeds the levels reported for MAPseq (up to 30% more accurate than existing solutions)
- Classifying reads against the default reference requires only 'mapseq rawseqs.fa > rawseqs.fa.mseq' with no extra index arguments, confirming the index and data files resolve relative to the binary path
- A '-otutable' run across multiple .mseq samples yields a TSV with samples as columns and OTU/taxonomic labels as rows at the '-ti'/'-tl' levels requested
- Reference sets reduced by removing near-identical sequences should show measurably lower F½ accuracy, matching the reported redundancy-accuracy trade-off

## Limitations

- Moving only the mapseq binary elsewhere breaks the installation, as data files are searched for relative to the binary's path
- Reducing reference dataset redundancy by removing near-identical sequences has a significant trade-off cost in mapping accuracy for all methods tested
- The '-ignoreEmptyTax' option (v1.2.4) defaults to off because thorough benchmarks were not performed; second hits with missing taxonomic labels can lower confidence of the top-hit assignment
- Version history notes earlier releases had a multithreaded race condition (fixed in 1.2.2) and double hits on long queries >1200bp (fixed in 1.2.3); use v1.2.6

## Evidence

- [abstract] MAPseq abstract: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score)"
- [readme] MAPseq README classification call: [evidence span withheld — anchor status: external_doc]
- [readme] MAPseq README custom reference: [evidence span withheld — anchor status: external_doc]
- [readme] MAPseq README data-file path constraint: [evidence span withheld — anchor status: external_doc]
- [readme] Confidence cutoff recommendation: [evidence span withheld — anchor status: external_doc]
