---
name: rrna-amplicon-taxonomic-classification
description: Use when demultiplexed, quality-filtered rRNA reads (amplicon or shotgun)
  need taxonomy and hierarchical OTU assignments with per-level confidences, especially
  on large datasets where runtime matters.
when_to_use_negative:
- Input is already a count/abundance table or OTU table — MAPseq classifies raw reads;
  use its -otutable aggregation on existing .mseq files instead of re-running classification
- Sequences are not ribosomal RNA reads — MAPseq is designed for rRNA sequences against
  a full-length reference rRNA database with precomputed OTU clusters
- A de novo OTU clustering is required without any reference database — MAPseq is
  a reference-based classifier and cannot cluster reads without references
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0637
tools:
- name: MAPseq
  role: 'Primary classifier: k-mer search with confidence estimates assigning taxonomy
    and hierarchical OTU mappings to rRNA reads'
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Generates the './configure' script used to build MAPseq from source
- name: GNU make
  role: Compiles and installs MAPseq ('make', 'make install', optional 'make check'
    self-tests)
- name: gcc
  role: C compiler used to build the MAPseq binaries
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_001
  - task_003
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:cd76b9fd5ca5bfde1f1415c0342e0c5f518948e2f1bcad230aa11726a4f68283
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - GNU make
  - MAPseq
  - autoconf
  - gcc
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# rrna-amplicon-taxonomic-classification

## Summary

Classify demultiplexed rRNA amplicon or shotgun reads against a full-length reference rRNA database using MAPseq v1.2.6, which returns NCBI taxonomy and hierarchical OTU (hOTU) assignments with per-level confidence estimates in a single run. It is chosen because it is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions.

## When to use

Use when demultiplexed, quality-filtered rRNA reads (amplicon or shotgun) need taxonomy and hierarchical OTU assignments with per-level confidences, especially on large datasets where runtime matters.

## When NOT to use

- Input is already a count/abundance table or OTU table — MAPseq classifies raw reads; use its -otutable aggregation on existing .mseq files instead of re-running classification
- Sequences are not ribosomal RNA reads — MAPseq is designed for rRNA sequences against a full-length reference rRNA database with precomputed OTU clusters
- A de novo OTU clustering is required without any reference database — MAPseq is a reference-based classifier and cannot cluster reads without references

## Inputs

- Demultiplexed, quality-filtered rRNA reads in FASTA format (e.g. rawseqs.fa)
- Default bundled reference rRNA database and taxonomy (or a custom nucleotide reference FASTA plus tab-separated taxonomy file(s) with '#cutoff:' and '#levels:' header lines)
- Previously generated .mseq output files for -otucounts / -otutable aggregation

## Outputs

- .mseq classification file with per-read reference hit, alignment statistics, and taxonomy/OTU assignments with per-level confidence values
- Per-sample taxonomy and OTU count summary (mapseq -otucounts)
- Multi-sample OTU/taxa count table in TSV format (mapseq -otutable), importable into R

## How to apply

Build MAPseq v1.2.6 from https://github.com/jfmrod/mapseq with the GNU autotools toolchain: run './bootstrap' if cloning from the repository, then './configure' (optionally './configure --prefix=$HOME/usr' for a user-local install), 'make', and 'make install'. Feed a demultiplexed, quality-filtered FASTA (derived from FASTQ) into 'mapseq rawseqs.fa > rawseqs.fa.mseq' to classify against the bundled default reference (NCBI taxonomy plus precomputed OTU clusters); alternatively supply a custom reference FASTA and one or more tab-separated taxonomy files whose header declares identity cutoffs and levels (e.g. '#cutoff: 0.00:0.08 ...' and '#levels: Kingdom Phylum Class Order Family Genus Species'). Use '--outfmt confidences' to obtain per-level combined confidences (score confidence plus identity-cutoff confidence per level) and apply the recommended combined confidence cutoff of 0.4 or 0.5, which gave the highest F½-score in the authors' benchmarks. Aggregate results with 'mapseq -otucounts sample1.mseq' for single-sample summaries or 'mapseq -otutable sample1.mseq sample2.mseq ...' for a TSV count table across samples, importable into R via read.table with sep='\t', header=TRUE. Judge success by well-formed .mseq lines (query id, reference id, alignment score, pairwise identity, matches/mismatches/gaps, coordinates, then taxonomy levels with confidences in [0,1]) and by agreement with the paper's benchmark expectation that MAPseq maximizes F½ around 0.86 (genus) and 0.96 (98%-identity hOTUs).

## Related tools

- **MAPseq** (Primary classifier: k-mer search with confidence estimates assigning taxonomy and hierarchical OTU mappings to rRNA reads) — https://github.com/jfmrod/mapseq
- **autoconf** (Generates the './configure' script used to build MAPseq from source)
- **GNU make** (Compiles and installs MAPseq ('make', 'make install', optional 'make check' self-tests))
- **gcc** (C compiler used to build the MAPseq binaries)

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- Output .mseq lines contain query id, reference id, alignment score, pairwise identity, matches/mismatches/gaps, alignment coordinates, strand, followed by taxonomy classifications with confidences, all confidence values bounded in [0,1]
- Combined confidence values fall near or below the recommended operating cutoffs of 0.4–0.5, which the authors report yielded the highest F½-score
- 'mapseq -otucounts sample1.mseq' returns a summary whose total counts match the read count in the .mseq header line (e.g. '#sample.mseq 102301')
- 'mapseq -otutable' on multiple .mseq files yields a TSV with samples as columns and OTU/taxa labels as rows, readable in R via read.table(sep='\t', header=TRUE)
- On known-identity benchmark reads, genus-level F½ approaches ~0.86 and 98%-identity hOTU F½ approaches ~0.96, consistent with the paper's reported maxima

## Limitations

- The mapseq binary must stay with its data directory: 'Moving only the binary elsewhere will break the installation though, as the data files are searched for in relation to the binary's path.'
- LTP taxonomy was dropped from the default reference 'due to low coverage', and the -ignoreEmptyTax option remains off by default 'until thorough benchmarks are performed'
- Accuracy depends on reference coverage: the identity-cutoff confidence exists because a query can be misclassified 'due to the inexistence of a sequence representative in the database of the true classification'

## Evidence

- [readme] MAPseq is a set of fast and accurate sequence read classification tools designed to assign taxonomy and OTU classifications to ribosomal RNA sequences.: [evidence span withheld — anchor status: external_doc]
- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [readme] This will classify all the sequences found in rawseqs against the standard reference dataset provided with MAPseq.: [evidence span withheld — anchor status: external_doc]
- [abstract] providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings: "providing in a single run multiple taxonomy classifications and hierarchical operational taxonomic unit mappings, for rRNA sequences"
