---
name: pcr-primer-mismatch-tolerance-handling
description: Use when benchmarking a taxonomic classifier on query fragments that
  mimic primer-trimmed metabarcoding reads, and you need to retrieve primer-anchored
  sequences from a reference/training FASTA with a defined primer mismatch tolerance.
when_to_use_negative:
- Input sequences are raw untrimmed metabarcoding reads that already contain primer
  sequences — here you would trim with CUTADAPT directly rather than retrieving primer-anchored
  training sequences
- The downstream analysis does not require primer-anchored fragments (e.g. full-length
  500 bp+ classifier evaluation, where the 70% genus bootstrap cutoff applies instead
  of the 60% cutoff for 200 bp fragments)
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: CUTADAPT
  role: Retrieves primer-anchored sequences from the training set FASTA by primer
    matching with up to 10% primer-sequence mismatch allowed
- name: Mesquite
  role: Aligns compiled CO1 primers against the Drosophila yakuba CO1 region (X03240)
    to determine primer binding regions
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:df9a3bfb971e4788411fe9d21111896b11863eea258c2d40d0971913a642d0b0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CUTADAPT
  - Mesquite
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# pcr-primer-mismatch-tolerance-handling

## Summary

Simulates how raw metabarcoding reads are primer-trimmed by allowing up to 10% mismatch in the primer sequence when retrieving primer-anchored fragments (via CUTADAPT) from a full-length CO1 training set, then truncating hits to 200 bp Illumina-read-length fragments for classifier benchmarking.

## When to use

Use when benchmarking a taxonomic classifier on query fragments that mimic primer-trimmed metabarcoding reads, and you need to retrieve primer-anchored sequences from a reference/training FASTA with a defined primer mismatch tolerance.

## When NOT to use

- Input sequences are raw untrimmed metabarcoding reads that already contain primer sequences — here you would trim with CUTADAPT directly rather than retrieving primer-anchored training sequences
- The downstream analysis does not require primer-anchored fragments (e.g. full-length 500 bp+ classifier evaluation, where the 70% genus bootstrap cutoff applies instead of the 60% cutoff for 200 bp fragments)

## Inputs

- CO1 training set FASTA of full-length 500 bp+ screened reference sequences (from the CO1Classifier repository training files)
- Compiled list of CO1 PCR primer sequences with literature references (Table S14)
- Reference CO1 region for primer mapping: Drosophila yakuba CO1, GenBank accession X03240
- Optional secondary-structure reference context: Bos taurus CO1, UniProt P00396

## Outputs

- primer_trimmed_200bp.fasta — primer-anchored 200 bp query fragments for downstream leave-one-out and cross-validation classifier benchmarks
- Primer binding-region annotations mapped onto the CO1 barcoding region

## How to apply

First compile the CO1 primers of interest (e.g. from Table S14) and align them against the Drosophila yakuba CO1 region (GenBank X03240) using Mesquite v3.10 to determine primer binding regions, with Bos taurus CO1 secondary structure (UniProt P00396) as reference context. Then load the CO1 training set FASTA (full-length 500 bp+ screened sequences from the CO1Classifier repository) and run CUTADAPT v1.10 with default settings, allowing up to a 10% mismatch in the primer sequence, to retrieve primer-trimmed sequences the same way real raw metabarcoding reads would be processed. Finally truncate each retrieved primer-anchored sequence to a 200 bp fragment — the average length of an Illumina read after primer trimming — and write the fragments to primer_trimmed_200bp.fasta. The 10% mismatch tolerance matters because GenBank submissions are often already primer-trimmed, so primers near the 5'/3' ends of the barcoding region (e.g. the Folmer primers) have low coverage among primer-anchored training sequences.

## Related tools

- **CUTADAPT** (Retrieves primer-anchored sequences from the training set FASTA by primer matching with up to 10% primer-sequence mismatch allowed)
- **Mesquite** (Aligns compiled CO1 primers against the Drosophila yakuba CO1 region (X03240) to determine primer binding regions)

## Examples

```
cutadapt -g CCCGGTACACTTTATTTTT -e 0.1 --discard-untrimmed CO1_training.fasta | python truncate.py --length 200 > primer_trimmed_200bp.fasta
```

## Evaluation signals

- All output records in primer_trimmed_200bp.fasta are exactly 200 bp primer-anchored fragments
- Every retrieved fragment was matched with no more than 10% mismatch in the primer sequence
- Primer-anchored fragments are under-represented at binding positions near the 5' and 3' ends of the barcoding region (e.g. Folmer primers), reflecting primer trimming in GenBank submissions
- The fragment set reproduces the input used for the paper's leave-one-out and cross-validation benchmarks of primer-anchored 200 bp sequences

## Limitations

- Because primers are often trimmed before sequences are submitted to GenBank, primer-anchored 200 bp queries anchored near the 5' and 3' ends of the barcoding region (including the Folmer primers) are under-represented in the training set, biasing downstream benchmarking at those positions
- Table S14 (primers tested and their references) is referenced but not included in the article section, so the full primer list must be sourced from the supplementary material

## Evidence

- [methods] Run CUTADAPT v1.10 with default settings, allowing up to a 10% mismatch in the primer sequence, to retrieve primer-trimmed sequences from the training set: [evidence span withheld — anchor status: not_found]
- [results] Because primers are often trimmed before sequences are submitted to GenBank, the Folmer barcoding primers and other primers designed near the 5′ and 3′ ends: [evidence span withheld — anchor status: not_found]
- [methods] Truncate each retrieved primer-anchored sequence to a 200 bp fragment to simulate the average length of an Illumina read after primer trimming.: [evidence span withheld — anchor status: not_found]
- [readme] Genus | 0.3 | 0.3 | 0.3 | 0.3 | 0.3: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
