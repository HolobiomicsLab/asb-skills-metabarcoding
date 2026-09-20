---
name: coi-primer-anchored-sequence-trimming
description: 'Use when benchmarking a CO1 taxonomic classifier on fragments that mimic
  real Illumina metabarcode reads: you need primer-anchored 200 bp query sequences
  derived from full-length training-set FASTA records rather than arbitrary truncations.'
when_to_use_negative:
- Input consists of already-trimmed raw amplicon reads from a real sequencing run
  — this skill builds simulated query fragments from full-length training references,
  not processing live data
- Sequences are shorter than 500 bp or contain nucleotide ambiguities, since the CO1
  training sets were filtered to 500bp+ ambiguity-free records
- The goal is actual taxonomic assignment of reads (use the trained RDP classifier
  directly) rather than constructing the benchmark fragment sets
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0632
tools:
- name: CUTADAPT
  role: Primer trimming of CO1 training-set sequences with default settings and up
    to 10% primer-sequence mismatch to retrieve primer-anchored sequences
- name: Mesquite
  role: Aligning literature-compiled CO1 primers against the Drosophila yakuba CO1
    region (GenBank X03240) to determine primer binding regions
- name: CO1Classifier training files
  role: Source of the full-length 500 bp+ CO1 training-set FASTA that the primer-anchored
    fragments are derived from
  repo: https://github.com/terrimporter/CO1Classifier/releases
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:8c7741b226c6afaa2d76d31d889b1fbb3ed698efcaad784eba57f52e7b008a2c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CO1Classifier training files
  - CUTADAPT
  - Mesquite
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# coi-primer-anchored-sequence-trimming

## Summary

Simulates real metabarcode query reads by using CUTADAPT to trim CO1 training-set sequences at primer binding sites (≤10% primer mismatch allowed) and truncating retrieved sequences to 200 bp primer-anchored fragments. This generates the primer-anchored 200 bp benchmark fragments used to test RDP classifier performance and reveals position-specific coverage gaps.

## When to use

Use when benchmarking a CO1 taxonomic classifier on fragments that mimic real Illumina metabarcode reads: you need primer-anchored 200 bp query sequences derived from full-length training-set FASTA records rather than arbitrary truncations.

## When NOT to use

- Input consists of already-trimmed raw amplicon reads from a real sequencing run — this skill builds simulated query fragments from full-length training references, not processing live data
- Sequences are shorter than 500 bp or contain nucleotide ambiguities, since the CO1 training sets were filtered to 500bp+ ambiguity-free records
- The goal is actual taxonomic assignment of reads (use the trained RDP classifier directly) rather than constructing the benchmark fragment sets

## Inputs

- CO1 training set FASTA of full-length (500 bp+) screened reference sequences from the CO1Classifier repository training files (version-ref release)
- Compiled CO1 primer list from the literature (Table S14)
- Drosophila yakuba CO1 reference sequence, GenBank accession X03240, for mapping primer binding regions

## Outputs

- primer_trimmed_200bp.fasta — one 200 bp primer-anchored fragment per retrieved training sequence, anchored at a primer binding position
- Primer-anchored 200 bp fragment set used for leave-one-out and cross-validation classifier benchmarks (with 60% bootstrap support cutoff at genus rank for 200 bp sequences)

## How to apply

First compile the CO1 primers from the literature (Table S14) and align them against the Drosophila yakuba CO1 region (GenBank X03240) in Mesquite v3.10 to determine primer binding positions. Load the CO1 training set FASTA (full-length 500 bp+ screened sequences from the CO1Classifier repository training files) and run CUTADAPT v1.10 with default settings, allowing up to 10% mismatch in the primer sequence, to retrieve primer-trimmed sequences exactly as raw metabarcode reads would be processed. Truncate each retrieved primer-anchored sequence to 200 bp to simulate the average Illumina read length after primer trimming, then write the fragments to primer_trimmed_200bp.fasta. The rationale is that primer-anchored placement determines which region of the CO1 barcoding region the fragment covers, and primers near the 5′/3′ ends (e.g., Folmer primers) yield fragments whose anchor positions are under-represented because primers are often trimmed before GenBank submission.

## Related tools

- **CUTADAPT** (Primer trimming of CO1 training-set sequences with default settings and up to 10% primer-sequence mismatch to retrieve primer-anchored sequences)
- **Mesquite** (Aligning literature-compiled CO1 primers against the Drosophila yakuba CO1 region (GenBank X03240) to determine primer binding regions)
- **CO1Classifier training files** (Source of the full-length 500 bp+ CO1 training-set FASTA that the primer-anchored fragments are derived from) — https://github.com/terrimporter/CO1Classifier/releases

## Examples

```
cutadapt -e 0.1 -g PRIMER_SEQ_FROM_TABLE_S14 -o primer_trimmed_200bp.fasta CO1_training.fasta
```

## Evaluation signals

- Every output record in primer_trimmed_200bp.fasta is exactly 200 bp long
- Each fragment was retrieved with no more than 10% mismatch in the primer sequence (CUTADAPT error rate 0.1)
- Each fragment is anchored at a known primer binding position mapped via Mesquite against X03240
- Fragments anchored at the Folmer and other 5′/3′-proximal primer positions are under-represented, matching the paper's finding about primer trimming before GenBank submission
- The fragment set reproduces the basis for the 60% genus-rank bootstrap cutoff reported for primer-anchored 200 bp sequences in Table 2

## Limitations

- Primers are often trimmed before sequences are submitted to GenBank, so Folmer barcoding primers and other 5′/3′-proximal primers have especially low coverage in the training set, biasing benchmark coverage for those anchor positions
- Species-rank assignment is not achievable with 200 bp fragments — the cutoff tables report NA for species with the CO1 Eukaryote v1 training set
- No species-rank cutoff guarantees 99% correct assignments; genus rank is the practical benchmark level for 200 bp fragments

## Evidence

- [results] Primer-anchored 200 bp sequences with 60% bootstrap cutoff at genus rank: [evidence span withheld — anchor status: not_found]
- [abstract] Classification performance across the barcoding region: "we show that classification performance is similar across the length of the CO1 barcoding region"
- [readme] Training files availability from CO1Classifier repo: "The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file) are available as 'version-ref'."
- [readme] Training set 500bp+ filter: "GenBank sequences were filtered to only include sequences 500bp+, containing no nucleotide ambiguities, and preferrably with a Linnean binomial species name"
