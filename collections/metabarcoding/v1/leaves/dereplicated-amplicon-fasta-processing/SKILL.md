---
name: dereplicated-amplicon-fasta-processing
description: 'Use when preparing raw paired-end amplicon reads for swarm clustering:
  input must be pair-assembled, trimmed, strictly dereplicated, and written as a linearized
  FASTA where each unique sequence label ends with an abundance count.'
when_to_use_negative:
- Input is still raw FASTQ — the article's pipeline requires pair-assembly and dereplication
  before clustering, not direct clustering
- You intend to emulate a 97% global-similarity threshold by converting it to a large
  d value — the article explicitly warns this is not a correct use of swarm
- Input is already a filtered occurrence table or a feature/OTU table rather than
  sequence data
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: swarm v3
  role: Consumes the dereplicated, abundance-annotated FASTA for fastidious d=1 clustering;
    strict dereplication of input is mandatory in swarm 3.0
  repo: https://github.com/torognes/swarm
- name: vsearch
  role: Used to properly pair-assemble SFF or FASTQ files before dereplication and
    clustering
  repo: https://github.com/torognes/vsearch
- name: cutadapt
  role: Trims adaptors and primers from amplicon reads prior to dereplication
- name: swarm-tests
  role: Unit tests (input formats, options, regressions) to validate swarm behaviour
    on prepared inputs
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btab493
    title: 'Swarm v3: towards tera-scale amplicon clustering'
  source_content_hash: sha256:d4b50526e91d05641300475ef30b671c3116163574404373168e29607a86af9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/swarm
  tools:
  - cutadapt
  - swarm v3
  - swarm-tests
  - vsearch
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 6
---

# dereplicated-amplicon-fasta-processing

## Summary

Prepare amplicon sequences into the strictly dereplicated, abundance-annotated FASTA format that swarm requires: identical reads are merged, sorted by decreasing abundance, and each entry label carries its occurrence count. This is a mandatory precondition for swarm clustering (d = 1) and for any swarm v2 vs v3 benchmarking on SSU-rRNA V4/V9 datasets.

## When to use

Use when preparing raw paired-end amplicon reads for swarm clustering: input must be pair-assembled, trimmed, strictly dereplicated, and written as a linearized FASTA where each unique sequence label ends with an abundance count.

## When NOT to use

- Input is still raw FASTQ — the article's pipeline requires pair-assembly and dereplication before clustering, not direct clustering
- You intend to emulate a 97% global-similarity threshold by converting it to a large d value — the article explicitly warns this is not a correct use of swarm
- Input is already a filtered occurrence table or a feature/OTU table rather than sequence data

## Inputs

- Raw paired-end SFF or FASTQ amplicon reads (e.g. SSU-rRNA V4 380 bp or V9 130 bp amplicons)
- Adaptor/primer-trimmed reads awaiting dereplication

## Outputs

- Dereplicated, abundance-annotated amplicon FASTA (unique sequences sorted by decreasing abundance, labels ending in occurrence counts such as _1000)
- Optionally SHA-1-relabeled, linearized FASTA ready as direct swarm input

## How to apply

First pair-assemble SFF/FASTQ files (e.g. with vsearch) and trim adaptors and primers (e.g. with cutadapt). Then perform dereplication: identify, merge, and sort identical sequences by decreasing abundance, since strict dereplication of input sequences is mandatory in swarm 3.0 and swarm requires each FASTA entry to carry a number of occurrences (e.g. labels like seqID1_1000) to work properly. Linearize the FASTA (--fasta_width 0) so each sequence occupies one line, and optionally relabel entries with SHA-1 hash values (--relabel_sha1) for stable identifiers. Feed the resulting dereplicated FASTA directly into swarm with the default local threshold d = 1 and the recommended --fastidious option for refinement; note that swarm's memory footprint is roughly 0.6x the input FASTA size, and can increase with fastidious (controlled via -c and -y).

## Related tools

- **swarm v3** (Consumes the dereplicated, abundance-annotated FASTA for fastidious d=1 clustering; strict dereplication of input is mandatory in swarm 3.0) — https://github.com/torognes/swarm
- **vsearch** (Used to properly pair-assemble SFF or FASTQ files before dereplication and clustering) — https://github.com/torognes/vsearch
- **cutadapt** (Trims adaptors and primers from amplicon reads prior to dereplication)
- **swarm-tests** (Unit tests (input formats, options, regressions) to validate swarm behaviour on prepared inputs) — https://github.com/torognes/swarm

## Examples

```
./swarm --fastidious --threads 4 --seeds cluster_representatives.fasta amplicons.fasta
```

## Evaluation signals

- Every FASTA entry label is unique and ends with an occurrence count (e.g. seqID1_1000); swarm aborts or misbehaves without abundance annotations
- Sequences are strictly dereplicated (no duplicate sequences remain) and sorted by decreasing abundance
- FASTA is linearized (--fasta_width 0) with one sequence per line, and labels optionally match --relabel_sha1 hash values
- Downstream swarm clustering with d=1 and --fastidious runs successfully and memory usage tracks ~0.6x the input FASTA size (plus fastidious overhead bounded via -c/-y)
- Prepared files pass swarm-tests input-format tests (bash run_all_tests.sh)

## Limitations

- Swarm requires each FASTA entry to present a number of occurrences; alternative label formats need the -z option
- When using the fastidious option, memory footprint can increase significantly and must be capped via the -c and -y options
- The default d = 1 may fail to distinguish molecular taxa when a marker does not evolve fast enough, producing large-radius clusters; higher d values (2-3) are only for long amplicons or shallow sequencing

## Evidence

- [methods] The role of the dereplication step is to identify, merge and sort identical sequences by decreasing abundance.: "The role of the dereplication step is to identify, merge and sort identical sequences by decreasing abundance."
- [readme] Swarm requires each fasta entry to present a number of occurrences to work properly.: "Swarm **requires** each fasta entry to present a number of occurrences to work properly."
- [readme] strict dereplication of input sequences is now mandatory: "strict dereplication of input sequences is now **mandatory**,"
- [methods] your SFF or FASTQ files have been properly pair-assembled (with vsearch for example): "your SFF or FASTQ files have been properly pair-assembled (with [vsearch](https://github.com/torognes/vsearch) for example)"
- [methods] the renaming with hashing values (--relabel_sha1): "the renaming with hashing values (`--relabel_sha1`)"
