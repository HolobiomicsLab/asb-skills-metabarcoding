---
name: naive-bayesian-classifier-throughput-profiling
description: Use when a CO1 metabarcoding pipeline must choose between a naive Bayesian
  classifier and BLAST-based assignment and throughput (queries per minute) on large
  FASTQ-derived read sets is a deciding factor.
when_to_use_negative:
- You need species-rank assignments — the CO1 Eukaryote v1 genus-rank classifier has
  'NA' (no cutoff) available at species rank for 99% correct assignments, so speed
  benchmarking at genus level would not support species-level conclusions
- Your queries are full-length 500 bp+ CO1 barcodes rather than primer-trimmed ~200
  bp metabarcode fragments — the 0.60 genus bootstrap cutoff and speed figures were
  established for 200 bp primer-anchored fragments
- You have already completed taxonomic assignment and only need to interpret or re-annotate
  an existing classification table
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: RDP classifier
  role: Naive Bayesian classifier run (v2.12, single processor) against the CO1 Eukaryote
    v1 training set to time genus-rank assignment throughput
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Top-hit comparison method, run against a locally installed copy of the NCBI
    nucleotide database to time assignment throughput on the same 200 bp queries
- name: CUTADAPT
  role: Retrieves primer-trimmed sequences (default settings, up to 10% primer mismatch)
    which are then cut to 200 bp fragments to simulate Illumina metabarcode reads
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:26f0fe2a0c84388973e6b6ac4a273d94c92a7431ef4e5e3c84c00d7d1d27381c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - CUTADAPT
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# Naive Bayesian classifier throughput profiling

## Summary

Benchmarks the speed of genus-rank taxonomic assignment of the RDP naive Bayesian classifier (CO1 Eukaryote v1 training set) against the top BLAST hit method on simulated Illumina CO1 reads, quantifying sequences-per-minute throughput for each method. The benchmark established that the RDP classifier assigns taxonomy about 19 times faster than BLAST on a single processor.

## When to use

Use when a CO1 metabarcoding pipeline must choose between a naive Bayesian classifier and BLAST-based assignment and throughput (queries per minute) on large FASTQ-derived read sets is a deciding factor.

## When NOT to use

- You need species-rank assignments — the CO1 Eukaryote v1 genus-rank classifier has 'NA' (no cutoff) available at species rank for 99% correct assignments, so speed benchmarking at genus level would not support species-level conclusions
- Your queries are full-length 500 bp+ CO1 barcodes rather than primer-trimmed ~200 bp metabarcode fragments — the 0.60 genus bootstrap cutoff and speed figures were established for 200 bp primer-anchored fragments
- You have already completed taxonomic assignment and only need to interpret or re-annotate an existing classification table

## Inputs

- CO1 Eukaryote v1 trained classifier files (rRNAClassifier.properties etc.) from github.com/terrimporter/CO1Classifier
- CO1 reference/training FASTA sequences for generating primer-trimmed 200 bp query fragments
- CUTADAPT v1.10 with CO1 primer sequences (up to 10% mismatch allowed)
- Locally installed copy of the NCBI nucleotide database for BLAST

## Outputs

- speed_benchmark.csv containing sequences-per-minute throughput for RDP classifier and top BLAST hit, and the RDP/BLAST speed ratio (~19x expected)
- Genus-rank RDP assignments for 200 bp fragments with bootstrap support values (good if >= 0.60)
- BLAST top-hit assignments with percent identity and alignment coverage (good if identity >= 95% and coverage >= 85% of query length)

## How to apply

First generate realistic query fragments: trim the CO1 training/reference sequences with CUTADAPT v1.10 (default settings, up to 10% primer-sequence mismatch) and cut them to 200 bp to simulate Illumina reads after primer trimming. Run the RDP classifier v2.12 (single processor) against the CO1 Eukaryote v1 training set (github.com/terrimporter/CO1Classifier), scoring genus-rank assignments as 'good' when the bootstrap proportion is >= 0.60, and record sequences classified per minute. Run the identical 200 bp queries against a locally installed copy of the NCBI nucleotide database with BLAST, scoring top hits as 'good' when percent identity >= 95% and alignment spans >= 85% of query length, and record sequences processed per minute. Compute the RDP/BLAST throughput ratio and write the comparison to speed_benchmark.csv; the result should reproduce the reported ~19x speed advantage of the RDP classifier.

## Related tools

- **RDP classifier** (Naive Bayesian classifier run (v2.12, single processor) against the CO1 Eukaryote v1 training set to time genus-rank assignment throughput) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Top-hit comparison method, run against a locally installed copy of the NCBI nucleotide database to time assignment throughput on the same 200 bp queries)
- **CUTADAPT** (Retrieves primer-trimmed sequences (default settings, up to 10% primer mismatch) which are then cut to 200 bp fragments to simulate Illumina metabarcode reads)

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query_200bp.fasta
```

## Evaluation signals

- speed_benchmark.csv exists and reports the RDP/BLAST throughput ratio within a range consistent with ~19x faster assignment
- Both methods were run on identical 200 bp primer-trimmed query fragments on a single processor
- RDP genus assignments counted as 'good' use the bootstrap proportion threshold of 0.60
- BLAST top hits counted as 'good' satisfy percent identity >= 95% AND alignment spanning >= 85% of the query length
- Throughput figures are expressed in sequences per minute for both methods before computing the ratio

## Limitations

- Speed comparison was measured with a single processor; parallel BLAST or multi-threaded RDP runs would change the ratio
- The ~19x figure applies to the CO1 Eukaryote v1 training set and 200 bp fragments; newer classifier versions (v5.x) have much larger reference sets (e.g. 2,216,547 COI sequences) and updated bootstrap cutoffs (Genus 0.3), which may alter both speed and cutoffs
- Fast classification does not address false negatives: sequences cannot be classified when congeneric references are missing from the database, a particular concern in quarantine situations
- No web portal is available for the RDP-based CO1 classification method; it is command-line only

## Evidence

- [abstract] the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method: "the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method"
- [results] The proportion of correct assignments of primer-anchored 200 bp sequences with and without 60% bootstrap support (Table 2) is also shown.: [evidence span withheld — anchor status: not_found]
- [discussion] the top BLAST hit method against a locally installed copy of the nucleotide database: "the top BLAST hit method against a locally installed copy of the nucleotide database"
- [readme] rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta: "rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta"
