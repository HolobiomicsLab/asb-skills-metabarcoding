---
name: fasta-roundtrip-validation
description: Use when implementing or porting memory-efficient DNA sequence storage
  (2-bit packing, Zobrist hashing, Bloom filters) and you must prove lossless encoding
  before trusting downstream clustering.
when_to_use_negative:
- Input sequences contain ambiguity/IUPAC codes beyond A/C/G/T, which cannot be represented
  in a 2-bit code without a lossy or extended scheme.
- The goal is clustering output validation (OTU structure), not sequence-storage fidelity
  — use the swarm-tests suite instead.
- Sequences are quality-carrying FASTQ reads; 2-bit packing here addresses nucleotide
  identity only, not qualities.
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0654
tools:
- name: swarm v3
  role: Reference implementation whose packed-sequence and Bloom-filter design (fastidious
    phase) the round-trip tests mirror
  repo: https://github.com/torognes/swarm
- name: GCC
  role: Compile the packing module (g++ version 4.8.5 or more recent)
- name: clang
  role: Alternative compiler (version 9 or more recent) for building the packing module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:9b291ddaebbc9519ff708bdc5d1d7d60e60533154daf724cbb3c1ad5640d4d4b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - clang
  - swarm v3
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0654
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.0
n_claims: 9
---

# FASTA round-trip validation of 2-bit packed sequence encoding

## Summary

Validates that a two-bit-per-nucleotide sequence compression (as used in swarm v3's packed-sequence representation) preserves DNA sequences bit-exactly by decoding every packed entry and comparing it against the original amplicon FASTA, while quantifying memory savings versus the 8 bits/nucleotide baseline.

## When to use

Use when implementing or porting memory-efficient DNA sequence storage (2-bit packing, Zobrist hashing, Bloom filters) and you must prove lossless encoding before trusting downstream clustering.

## When NOT to use

- Input sequences contain ambiguity/IUPAC codes beyond A/C/G/T, which cannot be represented in a 2-bit code without a lossy or extended scheme.
- The goal is clustering output validation (OTU structure), not sequence-storage fidelity — use the swarm-tests suite instead.
- Sequences are quality-carrying FASTQ reads; 2-bit packing here addresses nucleotide identity only, not qualities.

## Inputs

- Sample amplicon FASTA file (e.g. amplicons.fasta) with unique sequence identifiers carrying abundance suffixes (e.g. _1000)

## Outputs

- test_report.md with round-trip pass/fail per sequence, measured bits-per-nucleotide, and memory saving versus 8 bits/nucleotide
- Packed 2-bit sequence representation (fixed-length chunks) with Zobrist hash values inserted into a Bloom filter

## How to apply

Read an amplicon FASTA (entries carry unique identifiers ending in an abundance suffix, e.g. '>seqID1_1000', as required by swarm). Pack each nucleotide (A/C/G/T) into a 2-bit code, optionally compute Zobrist hashes from per-position random values and insert them into a Bloom filter sized for the hash set, mirroring swarm v3's fastidious-phase design on fixed-length chunks of compressed sequences. Compile the module with GCC (g++ 4.8.5+) or clang 9+, or run it directly if interpreted. Decode every packed sequence and verify bit-exact identity with the original input; then measure the packed representation's footprint and compute achieved bits per nucleotide. Acceptance requires all round-trip tests to pass and bits-per-nucleotide <= 2.25, confirming ~4x compression over one byte per nucleotide.

## Related tools

- **swarm v3** (Reference implementation whose packed-sequence and Bloom-filter design (fastidious phase) the round-trip tests mirror) — https://github.com/torognes/swarm
- **GCC** (Compile the packing module (g++ version 4.8.5 or more recent))
- **clang** (Alternative compiler (version 9 or more recent) for building the packing module)

## Examples

```
g++ -O2 -std=c++11 pack_and_roundtrip.cpp -o pack_and_roundtrip && ./pack_and_roundtrip --input amplicons.fasta --report test_report.md
```

## Evaluation signals

- Every decoded sequence is bit-exact identical to its original FASTA counterpart (all round-trip tests pass)
- Measured bits-per-nucleotide of the packed representation is <= 2.25
- Memory saving is ~4x relative to the 8 bits-per-nucleotide one-byte-per-base baseline
- Module builds cleanly with g++ >= 4.8.5 or clang >= 9
- Zobrist hashes for identical sequences are identical and distinct sequences do not collide in the Bloom filter (no false deduplication)

## Limitations

- Compression applies to A/C/G/T only; the 2-bit code cannot losslessly encode IUPAC ambiguity characters.
- Bloom filters are probabilistic: false positives are possible, so hash membership must be treated as approximate unless sized generously.
- Memory footprint of swarm grows significantly with the fastidious option; -c and -y options exist to cap memory consumption.

## Evidence

- [readme] where sequence identifiers are unique and end with a value indicating the number of occurrences of the sequence (e.g., `_1000`).: "where sequence identifiers are unique and end with a value indicating the number of occurrences of the sequence (e.g., `_1000`)."
- [abstract] swarm v3 reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [methods] The fastidious option is recommended when using d = 1, as it will reduce the number of small clusters: "The fastidious option is recommended when using *d* = 1, as it will reduce the number of small clusters"
- [readme] Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent): "Compiling `swarm` only requires `make`, and GCC (`g++` version 4.8.5 or more recent) or `clang` (version 9 or more recent)"
- [readme] The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file. When using the fastidious option, memory footprint can increase significantly.: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
