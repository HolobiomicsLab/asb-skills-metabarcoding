---
name: bloom-filter-hash-storage
description: Use when clustering or comparing massive amplicon datasets (tera-scale
  FASTA inputs) where storing one byte per nucleotide or full sequence objects exhausts
  memory, and fast approximate membership testing between amplicons is needed.
when_to_use_negative:
- Input is raw FASTQ reads or non-dereplicated FASTA — swarm requires strict dereplication
  first, with each entry carrying an occurrence count
- The task requires exact all-vs-all sequence alignment output; the Bloom filter only
  supports approximate membership screening, not full similarity computation
- Amplicons contain ambiguity codes beyond A/C/G/T, which have no 2-bit encoding
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3168
tools:
- name: swarm v3
  role: 'Reference implementation: 2-bit packed sequences in fixed-length chunks,
    Zobrist hashing and Bloom filter in the fastidious phase'
  repo: https://github.com/torognes/swarm
- name: GCC
  role: Compiler (g++ 4.8.5 or more recent) for building the packed-sequence module
    from source
- name: clang
  role: Alternative compiler (version 9 or more recent) for building the module
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:381524dcfda226de6faf2d6f95d8d28532e0f1c9c4132a2aa7086ae33ade5d77
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - GCC
  - clang
  - swarm v3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Bloom-filter hash storage

## Summary

A memory-reduction technique from swarm v3 that stores DNA amplicons as 2-bit packed sequences processed in fixed-length chunks, uses a fast Zobrist hash function to fingerprint them, and indexes the hash set in a Bloom filter to avoid storing full sequences — contributing to swarm v3's reduced memory footprint of up to 50%.

## When to use

Use when clustering or comparing massive amplicon datasets (tera-scale FASTA inputs) where storing one byte per nucleotide or full sequence objects exhausts memory, and fast approximate membership testing between amplicons is needed.

## When NOT to use

- Input is raw FASTQ reads or non-dereplicated FASTA — swarm requires strict dereplication first, with each entry carrying an occurrence count
- The task requires exact all-vs-all sequence alignment output; the Bloom filter only supports approximate membership screening, not full similarity computation
- Amplicons contain ambiguity codes beyond A/C/G/T, which have no 2-bit encoding

## Inputs

- Dereplicated amplicon FASTA file (e.g. amplicons.fasta) with unique identifiers ending in an abundance suffix such as _1000
- Reference nucleotide sequences (A/C/G/T only)

## Outputs

- 2-bit packed sequence representation (memory-footprint-measured, ~0.25 bytes per nucleotide)
- Bloom filter holding the Zobrist hash set of sequences
- Round-trip test report (per-sequence pass/fail, measured bits-per-nucleotide, memory saving vs 8 bits/nt)

## How to apply

Encode each nucleotide (A/C/G/T) into a 2-bit code so a sequence of length L occupies L/4 bytes instead of L bytes, achieving roughly 4x compression (≤ 2.25 bits per nucleotide measured). Process packed sequences as fixed-length chunks rather than nucleotide-by-nucleotide, mirroring swarm v3's rewrite. Compute sequence fingerprints with a Zobrist hash built from per-position random values, which is faster than recomputing naive hashes on every comparison. Insert hash values into a Bloom filter sized to hold the hash set, as used in swarm's fastidious phase, so candidate matches can be screened with negligible memory instead of keeping every sequence in RAM. Validate by round-trip decoding every packed sequence and verifying bit-exact identity with the FASTA input, then measure bits-per-nucleotide against the 8-bit baseline.

## Related tools

- **swarm v3** (Reference implementation: 2-bit packed sequences in fixed-length chunks, Zobrist hashing and Bloom filter in the fastidious phase) — https://github.com/torognes/swarm
- **GCC** (Compiler (g++ 4.8.5 or more recent) for building the packed-sequence module from source)
- **clang** (Alternative compiler (version 9 or more recent) for building the module)

## Examples

```
git clone https://github.com/torognes/swarm.git && cd swarm/ && make CC="clang-9" CXX="clang++-9"
```

## Evaluation signals

- All round-trip tests pass: every decoded packed sequence is bit-exact identical to the original FASTA input sequence
- Measured memory footprint of the packed representation is ≤ 2.25 bits per nucleotide, confirming ~4x compression over the 8 bits/nucleotide baseline
- Overall pipeline memory footprint stays around 0.6 times the size of the input FASTA file (swarm's documented figure), rising only when --fastidious is used
- Zobrist hash values are consistent across repeated encodings of the same sequence (deterministic fingerprinting)
- swarm's own regression suite passes: bash run_all_tests.sh from frederic-mahe/swarm-tests

## Limitations

- With the fastidious option, memory footprint can increase significantly despite the packed representation; options -c and -y exist to cap swarm's memory consumption
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so the fast path is tuned for the default d = 1
- Bloom filters allow false positives, so hash membership is a screening step, not proof of sequence identity; exact verification still requires comparing packed chunks
- The approach requires modernized C++ source code compiled with GCC (g++ ≥ 4.8.5) or clang (≥ 9); BSD systems need gmake instead of make

## Evidence

- [abstract] swarm v3 reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [full_text] Swarm v3 deploys a faster Zobrist hash function and an efficient Bloom filter, and operates on fixed-length chunks of compressed sequences: [evidence span withheld — anchor status: not_found]
- [readme] The memory footprint of swarm is roughly 0.6 times the size of the input fasta file: "The memory footprint of **swarm** is roughly 0.6 times the size of the
input fasta file. When using the fastidious option, memory footprint
can increase significantly. See options `-c` and `-y` to"
- [readme] swarm 3.0 introduces a reduced memory footprint: "swarm 3.0 introduces:
* a much faster default algorithm,
* a reduced memory footprint,"
- [readme] Compiling swarm only requires make, and GCC (g++ version 4.8.5 or more recent) or clang (version 9 or more recent): "Compiling `swarm` only requires
`make`, and GCC (`g++` version 4.8.5 or more recent) or `clang`
(version 9 or more recent)"
