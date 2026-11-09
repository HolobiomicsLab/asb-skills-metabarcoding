---
name: memory-footprint-benchmarking
description: Use when a clustering pipeline for amplicon FASTA inputs is expected
  to exceed available RAM (tera-scale datasets), and you must verify that 2-bit packed
  representations actually deliver the documented memory reduction.
when_to_use_negative:
- The input FASTA entries lack the required abundance suffixes (e.g. _1000), since
  swarm requires each entry to carry an occurrence count to work properly
- The question is clustering quality or OTU delineation rather than memory/compression
  measurement (e.g. choosing the local threshold d)
- Comparing wall-clock speed across versions without controlling for the documented
  ~15% slowdown at d >= 2 on some datasets (e.g. 18SV9)
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: swarm v3
  role: Amplicon clustering program whose 2-bit packed-sequence, Zobrist-hash and
    Bloom-filter memory representation is benchmarked
  repo: https://github.com/torognes/swarm
- name: GCC
  role: Compiler (g++ 4.8.5 or more recent) used to build the packing/benchmark module
- name: clang
  role: Alternative compiler (version 9 or more recent) for building the module
- name: swarm-tests
  role: Unit-test suite (run_all_tests.sh) providing black-box regression tests supporting
    correctness of the tested build
  repo: https://github.com/frederic-mahe/swarm-tests
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:25cacc71446ea92eb52fe00913317ff46d60e62078a6ce68e723aee437787246
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - clang
  - swarm v3
  - swarm-tests
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# memory-footprint-benchmarking

## Summary

Measures and validates the RAM savings obtained from swarm v3's compressed sequence representation (2-bit nucleotide packing with Zobrist hashing and a Bloom filter) by benchmarking memory footprint against the one-byte-per-nucleotide baseline and the ~0.6x input-FASTA rule of thumb.

## When to use

Use when a clustering pipeline for amplicon FASTA inputs is expected to exceed available RAM (tera-scale datasets), and you must verify that 2-bit packed representations actually deliver the documented memory reduction.

## When NOT to use

- The input FASTA entries lack the required abundance suffixes (e.g. _1000), since swarm requires each entry to carry an occurrence count to work properly
- The question is clustering quality or OTU delineation rather than memory/compression measurement (e.g. choosing the local threshold d)
- Comparing wall-clock speed across versions without controlling for the documented ~15% slowdown at d >= 2 on some datasets (e.g. 18SV9)

## Inputs

- Dereplicated amplicon FASTA file with unique sequence identifiers ending in occurrence counts (e.g. >seqID1_1000)
- Packed-sequence and Bloom-filter module source code (swarm v3 fastidious-phase design)
- Baseline one-byte-per-nucleotide in-memory representation for comparison

## Outputs

- Measured peak memory footprint of the packed representation
- Achieved bits-per-nucleotide value compared to the 8-bit baseline
- test_report.md with round-trip pass/fail per sequence and computed memory saving
- Bit-exact decoded sequences from round-trip packing/unpacking tests

## How to apply

Build swarm v3 from source with GCC (g++ 4.8.5 or more recent) or clang (version 9 or more recent), since v3 has modernized C++ source code. Swarm v3 encodes each nucleotide (A/C/G/T) as a 2-bit code, computes Zobrist hashes from per-position random values, and inserts hash values into a Bloom filter sized to the hash set, operating on fixed-length chunks of compressed sequences rather than individual nucleotides. Read a sample dereplicated amplicon FASTA (e.g. amplicons.fasta with unique identifiers and abundance suffixes like _1000), run the clustering, and record peak memory; compare against the baseline where memory is roughly 0.6 times the size of the input FASTA file. When the --fastidious option is used, memory can increase significantly, so repeat the measurement with the -c and -y options applied to cap memory consumption. Report achieved bits-per-nucleotide (target <= 2.25, i.e. ~4x compression over the 8 bits/nucleotide baseline) and total memory saving (up to 50% vs. earlier versions).

## Related tools

- **swarm v3** (Amplicon clustering program whose 2-bit packed-sequence, Zobrist-hash and Bloom-filter memory representation is benchmarked) — https://github.com/torognes/swarm
- **GCC** (Compiler (g++ 4.8.5 or more recent) used to build the packing/benchmark module)
- **clang** (Alternative compiler (version 9 or more recent) for building the module)
- **swarm-tests** (Unit-test suite (run_all_tests.sh) providing black-box regression tests supporting correctness of the tested build) — https://github.com/frederic-mahe/swarm-tests

## Examples

```
git clone https://github.com/torognes/swarm.git && cd swarm/ && make && ./bin/swarm --fastidious -c 4G amplicons.fasta
```

## Evaluation signals

- Round-trip test: every packed sequence decoded from the sample FASTA is bit-exact identical to the original input
- Measured bits-per-nucleotide is <= 2.25, confirming ~4x compression over the one-byte-per-nucleotide baseline
- Peak memory stays near the documented baseline of roughly 0.6 times the size of the input FASTA file in non-fastidious runs
- Memory reduction of up to 50% relative to previous swarm versions is reproduced on the test dataset
- With --fastidious, adding the -c and -y caps demonstrably bounds memory consumption compared to an uncapped run

## Limitations

- When using the fastidious option, memory footprint can increase significantly and must be capped via options -c and -y
- The 0.6x-input-FASTA memory figure is a rough approximation and may not hold for all datasets or option combinations
- Compression gains assume the standard A/C/G/T alphabet; ambiguous IUPAC codes and very short sequences dilute the 2-bit packing benefit

## Evidence

- [abstract] reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [readme] memory roughly 0.6 times input FASTA; fastidious increases memory: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file. When using the fastidious option, memory footprint can increase significantly."
- [readme] See options -c and -y to control and cap swarm's memory consumption: "See options `-c` and `-y` to control and cap swarm's memory consumption."
- [methods] GCC or clang version requirements: "GCC (`g++` version 4.8.5 or more recent) or `clang`"
- [abstract] modernized C++ source code in swarm v3: "swarm v3 has modernized C++ source code"
- [intro] swarm handles massive sets of amplicons: "The purpose of **swarm** is to provide a novel clustering algorithm
that handles massive sets of amplicons."
