---
name: expected-vs-recovered-abundance-comparison
description: Use when an artificial 'mock' community amplicon dataset with a known
  expected taxonomic composition is available and you need to measure how faithfully
  rRNA classifiers recover that composition.
when_to_use_negative:
- Input is real environmental/clinical data with no known expected taxonomic composition
  — there is no ground truth to compare abundances against
- The goal is OTU-based (closed-reference clustering) rather than taxonomy-based abundance
  recovery — MAPseq's hierarchical OTU mapping is a separate analysis path
- Reads are non-rRNA sequences; MAPseq is designed for ribosomal RNA sequence classification
  only
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MAPseq
  role: Reference-based k-mer classifier with confidence estimates used to classify
    mock community reads and generate abundance count summaries; benchmarked against
    competing tools
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Generates the './configure' script used to build MAPseq from source before
    benchmarking
  repo: https://github.com/jfmrod/mapseq
- name: GNU make
  role: Drives compilation and installation ('make', 'make install') of MAPseq prior
    to running the mock community benchmark
  repo: https://github.com/jfmrod/mapseq
- name: gcc
  role: C compiler invoked through './configure' (e.g. CC="gcc ...") to compile MAPseq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:f6638e278674604b78e13722bcf65b51956f3eeb50d8894ceb56a81923847c07
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
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# expected-vs-recovered-abundance-comparison

## Summary

Benchmarking skill that classifies an artificial mock rRNA community dataset with MAPseq and competing tools, aggregates per-tool classified read counts into species/genus/family abundance profiles, and compares recovered vs expected abundances to quantify classification accuracy.

## When to use

Use when an artificial 'mock' community amplicon dataset with a known expected taxonomic composition is available and you need to measure how faithfully rRNA classifiers recover that composition.

## When NOT to use

- Input is real environmental/clinical data with no known expected taxonomic composition — there is no ground truth to compare abundances against
- The goal is OTU-based (closed-reference clustering) rather than taxonomy-based abundance recovery — MAPseq's hierarchical OTU mapping is a separate analysis path
- Reads are non-rRNA sequences; MAPseq is designed for ribosomal RNA sequence classification only

## Inputs

- Artificial mock community amplicon sequencing dataset (rRNA reads, e.g. FASTA/FASTQ) with known expected taxonomic composition (Supplementary Fig. S4)
- MAPseq reference dataset (default NCBI taxonomy and OTU references, downloaded by ./setup.sh) or custom reference FASTA plus taxonomy file(s)
- Classification outputs (.mseq files) from competing rRNA tools run on the same read set

## Outputs

- Per-tool abundance profiles at species, genus, and family taxonomic ranks
- Tool-by-level agreement metrics comparing recovered vs expected abundances (e.g. F½ score)
- Count summary table ('-otucounts') or multi-sample OTU/taxa table ('-otutable', TSV importable into R)

## How to apply

Build MAPseq from source (MAPseq v1.2.6 or later) using the standard GNU build system ('./configure', 'make', 'make install'), installing autotools/autoconf, svn, git, wget, libtool, and libncurses5-dev first, and running './setup.sh' and './bootstrap' if cloning the repository. Load the artificial mock community amplicon dataset (Supplementary Fig. S4) whose expected composition at species, genus, and family level is known. Classify the identical read set with MAPseq's reference-based k-mer search with confidence estimates and with competing rRNA tools, keeping inputs identical for a fair comparison. Apply a combined confidence cutoff of 0.4 or 0.5 (the value that yielded the highest F1/2-score in the authors' benchmarks) and aggregate classified reads into abundance profiles — e.g. via 'mapseq -otucounts' or 'mapseq -otutable' — at the species, genus, and family ranks. Compute agreement metrics between recovered and expected abundances per tool and per level, and verify MAPseq recovers expected mock community abundances better than competing solutions.

## Related tools

- **MAPseq** (Reference-based k-mer classifier with confidence estimates used to classify mock community reads and generate abundance count summaries; benchmarked against competing tools) — https://github.com/jfmrod/mapseq
- **autoconf** (Generates the './configure' script used to build MAPseq from source before benchmarking) — https://github.com/jfmrod/mapseq
- **GNU make** (Drives compilation and installation ('make', 'make install') of MAPseq prior to running the mock community benchmark) — https://github.com/jfmrod/mapseq
- **gcc** (C compiler invoked through './configure' (e.g. CC="gcc ...") to compile MAPseq)

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq; mapseq -otutable rawseqs.fa.mseq competitor1.mseq competitor2.mseq > mock.otutable
```

## Evaluation signals

- The tool-by-level comparison table shows MAPseq recovering expected mock community abundances better than competing tools at species, genus, and family levels
- Running 'mapseq -otucounts <sample1.mseq>' yields a well-formed counts table with header lines '#sample.mseq', 'Taxonomy', 'TaxonomyLevel', 'Label', 'Counts' and counts summing to the total classified reads
- 'mapseq -otutable' across all tools' .mseq files produces a TSV with samples as columns and taxonomic labels as rows that imports cleanly into R via read.table(...,sep="\t",header=TRUE)
- Applied confidence cutoff is 0.4 or 0.5, matching the combined-confidence threshold with the highest F½-score reported in the benchmark
- All competing tools were run on the identical read set, confirming the comparison is fair

## Limitations

- Results depend on the confidence cutoff; values other than the recommended 0.4–0.5 reduce the F½-score in the authors' benchmarks
- MAPseq finds its data files relative to the binary's path, so moving only the binary breaks the installation
- The '-ignoreEmptyTax' option is off by default 'until thorough benchmarks are performed', so second hits with missing taxonomic labels can decrease the confidence of the top hit assignment
- Abundance recovery is only as good as the reference: the default reference must be present (via './setup.sh') or a custom reference FASTA and taxonomy files supplied

## Evidence

- [results] On artificial mock community data, MAPseq recovers expected abundances better than other tools, at the species, genus and family levels.: [evidence span withheld — anchor status: not_found]
- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [methods] ./configure
make
make install: "Type 'make' to compile the package."
- [readme] mapseq -otutable <sample1.mseq> <sample2.mseq> ...: [evidence span withheld — anchor status: external_doc]
