---
name: dna-sequence-two-bit-encoding
description: Use when memory-efficient storage of many amplicon sequences (e.g. FASTA
  dereplicated reads with abundance suffixes) is needed, or when implementing swarm
  v3-style fastidious clustering on compressed fixed-length chunks.
when_to_use_negative:
- Input is not dereplicated nucleotide FASTA — swarm requires strict dereplication
  and per-sequence occurrence counts, so raw FASTQ or non-dereplicated reads must
  be preprocessed first
- The task needs full sequence similarity computation (e.g. pairwise alignment) where
  the 2-bit alphabet cannot represent ambiguous IUPAC nucleotide codes
- Only a handful of short sequences are involved, where the packing machinery and
  Bloom filter overhead outweigh the memory savings
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: swarm v3
  role: 'Reference implementation: encodes nucleotides as 2-bit packed chunks, uses
    a faster Zobrist hash and an efficient Bloom filter in its fastidious phase'
  repo: https://github.com/torognes/swarm
- name: GCC
  role: Compiler for the packing module (g++ 4.8.5 or more recent)
- name: clang
  role: Alternative compiler for the packing module (version 9 or more recent)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:5b7d3f388055ac706c1a75c20ec5a65d40ffc0e6d1a3e2bbee52f6d183d16f28
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.14285714285714285
n_claims: 7
---

# dna-sequence-two-bit-encoding

## Summary

Packs DNA amplicon sequences by encoding each nucleotide (A/C/G/T) into a 2-bit code, computes Zobrist hashes from per-position random values, and indexes them in a Bloom filter, mirroring the packed-sequence design used in swarm v3's fastidious phase to cut the memory footprint (~4x compression vs 8 bits/nucleotide).

## When to use

Use when memory-efficient storage of many amplicon sequences (e.g. FASTA dereplicated reads with abundance suffixes) is needed, or when implementing swarm v3-style fastidious clustering on compressed fixed-length chunks.

## When NOT to use

- Input is not dereplicated nucleotide FASTA — swarm requires strict dereplication and per-sequence occurrence counts, so raw FASTQ or non-dereplicated reads must be preprocessed first
- The task needs full sequence similarity computation (e.g. pairwise alignment) where the 2-bit alphabet cannot represent ambiguous IUPAC nucleotide codes
- Only a handful of short sequences are involved, where the packing machinery and Bloom filter overhead outweigh the memory savings

## Inputs

- Dereplicated amplicon FASTA file (e.g. amplicons.fasta) with unique identifiers ending in abundance values (e.g. _1000)
- Set of sequences to pack (linearized FASTA, --fasta_width 0 formatting)

## Outputs

- Packed-sequence memory representation (2-bit encoded fixed-length chunks)
- Zobrist hash values stored in a Bloom filter
- Round-trip test report (test_report.md: per-sequence pass/fail, measured bits-per-nucleotide, memory saving vs 8 bits/nucleotide)

## How to apply

Read dereplicated amplicon FASTA entries whose identifiers end with an occurrence count (e.g. '>seqID1_1000'), then encode each nucleotide A/C/G/T into a 2-bit code and store sequences as fixed-length chunks of packed bits rather than individual nucleotides. Compute Zobrist hashes from per-position random values over the packed sequences and insert the hash values into a Bloom filter sized to hold the hash set, as swarm v3 does in its fastidious phase. Compile the module with GCC (g++ 4.8.5 or more recent) or clang (version 9 or more recent) per the swarm README. Validate by round-trip decoding every packed sequence and requiring bit-exact identity with the FASTA input, then measure achieved bits per nucleotide against the 8 bits-per-nucleotide baseline (target <= 2.25, i.e. ~4x compression).

## Related tools

- **swarm v3** (Reference implementation: encodes nucleotides as 2-bit packed chunks, uses a faster Zobrist hash and an efficient Bloom filter in its fastidious phase) — https://github.com/torognes/swarm
- **GCC** (Compiler for the packing module (g++ 4.8.5 or more recent))
- **clang** (Alternative compiler for the packing module (version 9 or more recent))

## Examples

```
git clone https://github.com/torognes/swarm.git; cd swarm/; make CC="clang-9" CXX="clang++-9"
```

## Evaluation signals

- All round-trip tests pass: every decoded packed sequence matches the reference FASTA input exactly
- Measured bits-per-nucleotide of the packed representation is <= 2.25
- Memory saving of roughly 4x relative to the 8-bits-per-nucleotide baseline is quantified
- Bloom filter is sized to hold the full Zobrist hash set (no overflow/fp-rate explosion) as in swarm's fastidious phase
- swarm's own memory footprint behavior is consistent: roughly 0.6x the size of the input FASTA file

## Limitations

- Swarm's memory footprint can increase significantly when using the fastidious option; options -c and -y control and cap memory consumption
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so the packed fastidious path is tuned for d = 1 use
- Ambiguous nucleotide codes beyond A/C/G/T are not directly representable in a pure 2-bit encoding

## Evidence

- [methods] encode each nucleotide (A/C/G/T) into a 2-bit code, compute Zobrist hashes from per-position random values, and insert hash values into a Bloom filter sized to hold the hash set: [evidence span withheld — anchor status: not_found]
- [methods] swarm v3 deploys a faster Zobrist hash function and an efficient Bloom filter, and was re-written to operate on fixed-length chunks of compressed sequences: [evidence span withheld — anchor status: not_found]
- [results] decoded sequences match the reference input exactly and bits-per-nucleotide <= 2.25: [evidence span withheld — anchor status: not_found]
- [abstract] reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [readme] The memory footprint of swarm is roughly 0.6 times the size of the input fasta file.: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
