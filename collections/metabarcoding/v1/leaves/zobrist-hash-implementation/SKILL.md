---
name: zobrist-hash-implementation
description: Use when implementing or porting swarm v3's memory-efficient amplicon
  representation (2-bit packed sequences, Zobrist hashing, Bloom filter) for the fastidious
  phase, or when a tera-scale amplicon FASTA exceeds available memory.
when_to_use_negative:
- Input is raw multi-FASTQ reads or un-dereplicated FASTA without abundance suffixes
  — swarm requires strict dereplication first, and identifiers must carry occurrence
  counts
- You need per-nucleotide random access or modification of sequences during analysis
  — the 2-bit packed representation is designed for bulk hashing/comparison, not editing
- The goal is global-threshold clustering (e.g. 97% OTU picking) — swarm instead uses
  a small local linking threshold d (default d = 1)
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: swarm v3
  role: Reference implementation whose fastidious phase deploys the faster Zobrist
    hash function, efficient Bloom filter, and 2-bit packed-sequence chunks being
    reimplemented
  repo: https://github.com/torognes/swarm
- name: GCC
  role: Compiler toolchain used to build the packed-sequence/hashing module (g++ 4.8.5
    or more recent)
- name: clang
  role: Alternative compiler for the module (clang version 9 or more recent)
- name: swarm-tests
  role: Black-box unit tests (806 tests in swarm 3.0's hardened suite; run_all_tests.sh)
    used as a template for regression and round-trip validation
  repo: https://github.com/frederic-mahe/swarm-tests
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:60996ec203f6691c91c1ab3df8cef4eb8a7d1443d996b5d4aa40810d8a905902
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
yes_rate: 0.3333333333333333
n_claims: 9
---

# zobrist-hash-implementation

## Summary

Implements the swarm v3 sequence-compression layer: encoding each DNA nucleotide (A/C/G/T) as a 2-bit code, computing Zobrist hashes from per-position random values, and inserting hash values into a Bloom filter. This reproduces the packed-sequence and Bloom-filter design that lets swarm v3 cut its memory footprint by up to 50% versus earlier versions.

## When to use

Use when implementing or porting swarm v3's memory-efficient amplicon representation (2-bit packed sequences, Zobrist hashing, Bloom filter) for the fastidious phase, or when a tera-scale amplicon FASTA exceeds available memory.

## When NOT to use

- Input is raw multi-FASTQ reads or un-dereplicated FASTA without abundance suffixes — swarm requires strict dereplication first, and identifiers must carry occurrence counts
- You need per-nucleotide random access or modification of sequences during analysis — the 2-bit packed representation is designed for bulk hashing/comparison, not editing
- The goal is global-threshold clustering (e.g. 97% OTU picking) — swarm instead uses a small local linking threshold d (default d = 1)

## Inputs

- Dereplicated amplicon FASTA file (e.g. amplicons.fasta) with unique sequence identifiers ending in an abundance value (e.g. '_1000')
- Per-position random Zobrist hash tables seeded for the target sequence length
- Bloom filter sized to hold the full hash set of amplicons

## Outputs

- 2-bit packed (compressed) sequence representation operating on fixed-length chunks
- Zobrist hash values per sequence stored in a Bloom filter
- test_report.md with round-trip pass/fail per sequence, measured bits-per-nucleotide, and memory savings versus the 8-bit baseline

## How to apply

Read the dereplicated amplicon FASTA (e.g. amplicons.fasta) whose unique identifiers carry abundance suffixes such as '_1000', as required by swarm's input format. Pack each sequence by encoding A/C/G/T into 2-bit codes, then compute Zobrist hashes by XOR-combining per-position random values, mirroring the fixed-length chunks of compressed sequences that swarm v3 operates on instead of individual nucleotides. Insert the hash values into a Bloom filter sized to hold the hash set, as used in swarm's fastidious phase to cheaply test for candidate nearby sequences. Compile the module with g++ 4.8.5 or more recent, or clang version 9 or more recent. Validate with round-trip tests: decode every packed sequence and require bit-exact identity with the original input, and measure achieved bits-per-nucleotide, which must be <= 2.25 versus the 8-bit-per-nucleotide baseline (~4x compression).

## Related tools

- **swarm v3** (Reference implementation whose fastidious phase deploys the faster Zobrist hash function, efficient Bloom filter, and 2-bit packed-sequence chunks being reimplemented) — https://github.com/torognes/swarm
- **GCC** (Compiler toolchain used to build the packed-sequence/hashing module (g++ 4.8.5 or more recent))
- **clang** (Alternative compiler for the module (clang version 9 or more recent))
- **swarm-tests** (Black-box unit tests (806 tests in swarm 3.0's hardened suite; run_all_tests.sh) used as a template for regression and round-trip validation) — https://github.com/frederic-mahe/swarm-tests

## Examples

```
make CC="clang-9" CXX="clang++-9"
```

## Evaluation signals

- All round-trip tests pass: decoding every packed sequence yields bit-exact identity with the reference FASTA input
- Measured representation cost is <= 2.25 bits per nucleotide, confirming ~4x compression over the one-byte-per-nucleotide baseline
- Memory footprint of the packed representation plus Bloom filter stays within swarm's documented envelope (roughly 0.6x the input FASTA size baseline; up to 50% overall reduction claimed for v3)
- Zobrist hashes are collision-free across the dereplicated input set (each unique amplicon gets a distinct hash); Bloom filter inserts/lookups return no false negatives
- Code compiles cleanly with g++ 4.8.5+ or clang 9+ and passes the swarm-tests suite (bash run_all_tests.sh)

## Limitations

- The fastidious option can increase swarm's memory footprint significantly; use options -c and -y to control and cap memory consumption
- A 15% slowdown was observed when using d >= 2 in certain conditions (18SV9 dataset), so the fast hashing/Bloom-filter path is tuned for d = 1
- The Bloom filter admits false positives by design; sequences failing round-trip or hash checks indicate packing or table-generation bugs, not acceptable compression artifacts

## Evidence

- [other] Swarm v3 deploys a faster Zobrist hash function and an efficient Bloom filter, and was re-written to operate on fixed-length chunks of compressed sequences instead of individual nucleotides.: [evidence span withheld — anchor status: not_found]
- [abstract] swarm v3 reduced memory footprint by up to 50%: "reduced memory footprint by up to 50%"
- [methods] The fastidious option is recommended when using d = 1, as it will reduce the number of small clusters: "The fastidious option is recommended when using *d* = 1, as it will reduce the number of small clusters"
- [readme] The memory footprint of swarm is roughly 0.6 times the size of the input fasta file.: "The memory footprint of **swarm** is roughly 0.6 times the size of the input fasta file."
- [readme] Compiling swarm only requires make, and GCC (g++ version 4.8.5 or more recent) or clang (version 9 or more recent): "Compiling `swarm` only requires
`make`, and GCC (`g++` version 4.8.5 or more recent) or `clang`
(version 9 or more recent)"
