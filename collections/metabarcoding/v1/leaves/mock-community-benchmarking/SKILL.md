---
name: mock-community-benchmarking
description: Use when you need to quantify how faithfully an rRNA amplicon classification
  tool recovers known taxon abundances, using an artificial mock community dataset
  with a known expected taxonomic composition as ground truth.
when_to_use_negative:
- Input data is a real environmental or clinical community with no known expected
  composition — there is no ground truth to compare recovered abundances against,
  so abundance-recovery accuracy cannot be computed
- The goal is speed or resource benchmarking only; while MAPseq reports 'up to one
  hundred times faster', this workflow is designed for abundance-recovery accuracy,
  not runtime profiling
- Input is already a summarized abundance/OTU table — classification and per-read
  confidence estimation are unnecessary and the benchmarking must start from raw classified
  reads
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3300
tools:
- name: MAPseq
  role: Reference-based rRNA read classifier under benchmark; classifies mock community
    reads and generates abundance count tables via -otucounts/-otutable
  repo: https://github.com/jfmrod/mapseq
- name: autoconf
  role: Build tool used to generate the 'configure' script when compiling MAPseq from
    source before benchmarking
  repo: https://github.com/jfmrod/mapseq
- name: GNU make
  role: Compiles and installs MAPseq ('make', 'make install') prior to running the
    mock community comparison
- name: gcc
  role: C compiler invoked by the GNU build system to compile MAPseq
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:eb1f6434f21d9eb70b8326fb058d21e939b70c798e52e17af0dd8d943c351ebf
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
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# mock-community-benchmarking

## Summary

Benchmark a taxonomy/OTU classification tool by classifying an artificial mock rRNA community with known expected composition and comparing recovered abundances against expected values at species, genus, and family levels. In the MAPseq article this workflow demonstrated MAPseq recovers expected mock community abundances better than competing tools.

## When to use

Use when you need to quantify how faithfully an rRNA amplicon classification tool recovers known taxon abundances, using an artificial mock community dataset with a known expected taxonomic composition as ground truth.

## When NOT to use

- Input data is a real environmental or clinical community with no known expected composition — there is no ground truth to compare recovered abundances against, so abundance-recovery accuracy cannot be computed
- The goal is speed or resource benchmarking only; while MAPseq reports 'up to one hundred times faster', this workflow is designed for abundance-recovery accuracy, not runtime profiling
- Input is already a summarized abundance/OTU table — classification and per-read confidence estimation are unnecessary and the benchmarking must start from raw classified reads

## Inputs

- Artificial mock community amplicon sequencing dataset (rRNA reads, e.g. FASTA/FASTQ) with known expected taxonomic composition at species, genus, and family level
- MAPseq reference dataset (default NCBI taxonomy plus OTU taxonomies, downloaded via ./setup.sh) or custom reference FASTA with taxonomy file
- Classification outputs (.mseq files) from each tool run on the identical read set
- Competing rRNA classification tool outputs for fair tool-by-tool comparison

## Outputs

- Per-tool abundance profiles aggregated at species, genus, and family taxonomic ranks (e.g. MAPseq -otucounts/-otutable count tables)
- Tool-by-level agreement metrics between recovered and expected mock community abundances (accuracy/F½ score comparisons)
- A single assembled comparison table showing whether the tool under test outperforms competing solutions

## How to apply

Obtain an artificial mock community amplicon sequencing dataset (e.g. the one underlying Supplementary Fig. S4 in the MAPseq article) for which the expected taxonomic composition at species, genus, and family level is known. Build and install the classification tool from source — for MAPseq (v1.2.6 or later) clone https://github.com/jfmrod/mapseq and run './bootstrap', './configure', 'make', 'make install'. Classify the identical mock community read set with MAPseq (reference-based k-mer search with confidence estimates) and with each competing rRNA classification tool, keeping the input FASTA and any confidence cutoff (MAPseq authors recommend a combined confidence cutoff of 0.4–0.5, which yielded the highest F1/2-score in their benchmarks) the same across tools for a fair comparison. Aggregate per-tool classified read counts into abundance profiles using MAPseq's '-otucounts' for a single .mseq file or '-otutable' for multiple samples, at species, genus, and family taxonomic ranks. Compute agreement metrics between recovered and expected abundances for each tool at each taxonomic level (MAPseq accuracy is reported as F½ score), and assemble a single tool-by-level comparison table. Verify the conclusion only if the metrics show the tool under test recovers expected abundances better than competitors, as MAPseq did (up to 30% more accurate and up to 100x faster).

## Related tools

- **MAPseq** (Reference-based rRNA read classifier under benchmark; classifies mock community reads and generates abundance count tables via -otucounts/-otutable) — https://github.com/jfmrod/mapseq
- **autoconf** (Build tool used to generate the 'configure' script when compiling MAPseq from source before benchmarking) — https://github.com/jfmrod/mapseq
- **GNU make** (Compiles and installs MAPseq ('make', 'make install') prior to running the mock community comparison)
- **gcc** (C compiler invoked by the GNU build system to compile MAPseq)

## Examples

```
git clone https://github.com/jfmrod/MAPseq.git; cd MAPseq; ./setup.sh; ./bootstrap; ./configure; make; make install; mapseq mock_reads.fa > mock_reads.fa.mseq; mapseq -otutable mock_reads.fa.mseq > mock.otutable
```

## Evaluation signals

- Recovered abundance profiles at species, genus, and family levels show higher agreement with the expected mock community composition for the tool under test than for each competing tool
- MAPseq's F½ score on the mock community exceeds that of competing solutions (article reports up to 30% more accurate F½ score)
- All tools are run on the identical read set with identical reference data and the same confidence cutoff (recommended combined confidence 0.4–0.5) so the comparison is fair
- Aggregated count tables are well-formed (e.g. tab-separated map.otutable importable into R with read.table(header=TRUE)) and counts sum consistently per sample
- MAPseq build completes cleanly through './configure', 'make', 'make install', confirming the version benchmarked is v1.2.6 or later

## Limitations

- Benchmark validity depends on the mock community's known expected composition (Supplementary Fig. S4 dataset); results do not transfer to communities without ground truth
- MAPseq confidence cutoffs matter: the authors recommend a combined confidence cutoff of 0.4 or 0.5, which yielded the highest F1/2-score in their benchmarks; other cutoffs may degrade accuracy
- History entries note version-specific issues (e.g. v1.2.2 fixed a multithreaded race condition; v1.2.4's -ignoreEmptyTax default is off 'until thorough benchmarks are performed'), so tool version can affect benchmark outcomes
- Moving only the mapseq binary elsewhere breaks the installation because data files are searched relative to the binary's path

## Evidence

- [results] On artificial mock community data, MAPseq recovers expected abundances better than other tools, at the species, genus and family levels.: [evidence span withheld — anchor status: not_found]
- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [methods] cd' to the directory containing the package's source code and type './configure' to configure the package for your system.: "cd' to the directory containing the package's source code and type './configure' to configure the package for your system."
- [readme] MAPseq v1.2.6 (24 Mar 2020): "MAPseq v1.2.6 (24 Mar 2020)"
