---
name: blast-top-hit-search-criteria
description: Use when CO1 metabarcode query fragments need taxonomic assignment by
  top BLAST hit, or when benchmarking assignment speed against the RDP classifier
  with explicit 'good' hit thresholds (identity >= 95%, coverage >= 85% of query).
when_to_use_negative:
- When you need probabilistic confidence scores (bootstrap support cutoffs at genus
  rank) rather than database similarity top hits — use the RDP classifier instead
- When the query sequences are not represented in the nucleotide reference database
  — the top BLAST hit method assumes the query sequence is actually present in the
  database
- When input is already a classified/feature table and the goal is summarization rather
  than raw sequence assignment
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: BLAST
  role: Performs the top-hit similarity search of 200 bp CO1 query fragments against
    the locally installed NCBI nucleotide database; also used for human contaminant
    screening with a custom human CO1 database
- name: CUTADAPT
  role: Used with default settings (up to 10% primer mismatch) to retrieve primer-trimmed
    sequences and trim them to 200 bp fragments simulating Illumina reads prior to
    BLAST classification
- name: RDP classifier
  role: 'Comparator method: RDP classifier v2.12 with the CO1 Eukaryote v1 training
    set (0.60 bootstrap cutoff at genus rank) benchmarked against the BLAST top-hit
    method for speed'
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:e1b32c7e4e77737959690449e194bb527cb0ca6824d7ade8b9eca9921efbbcb0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - CUTADAPT
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# blast-top-hit-search-criteria

## Summary

Assign taxonomy to CO1 metabarcode query sequences via the top BLAST hit method against a locally installed copy of the NCBI nucleotide database, defining 'good' assignments as percent identity >= 95% with alignment spanning >= 85% of the query length, and recording sequences-per-minute throughput for speed benchmarking against the RDP classifier.

## When to use

Use when CO1 metabarcode query fragments need taxonomic assignment by top BLAST hit, or when benchmarking assignment speed against the RDP classifier with explicit 'good' hit thresholds (identity >= 95%, coverage >= 85% of query).

## When NOT to use

- When you need probabilistic confidence scores (bootstrap support cutoffs at genus rank) rather than database similarity top hits — use the RDP classifier instead
- When the query sequences are not represented in the nucleotide reference database — the top BLAST hit method assumes the query sequence is actually present in the database
- When input is already a classified/feature table and the goal is summarization rather than raw sequence assignment

## Inputs

- FASTA file of 200 bp CO1 metabarcode query fragments (primer-trimmed, simulating Illumina reads)
- Locally installed copy of the NCBI nucleotide BLAST database

## Outputs

- BLAST top-hit assignments with percent identity and alignment coverage per query
- Sequences-per-minute throughput measurements for the top BLAST hit method
- Speed comparison table (speed_benchmark.csv) with the RDP/BLAST speed ratio

## How to apply

First generate primer-trimmed 200 bp query fragments (e.g. with CUTADAPT v1.10 allowing up to 10% primer mismatch) to simulate Illumina reads after primer trimming. Install a local copy of the NCBI nucleotide database and search the same 200 bp query fragments with BLAST, scoring each top hit as 'good' when percent identity >= 95% and the alignment spans >= 85% of the query length. Record the number of sequences processed per minute using a single processor to obtain comparable throughput. Compute sequences-per-minute for both BLAST and the RDP classifier v2.12 (CO1 Eukaryote v1 training set, 0.60 bootstrap cutoff at genus rank) and write the comparison to speed_benchmark.csv. The rationale is that top-hit assignment quality depends on the query being represented in the database, and that throughput differences are large enough to matter for high-throughput metabarcoding pipelines.

## Related tools

- **BLAST** (Performs the top-hit similarity search of 200 bp CO1 query fragments against the locally installed NCBI nucleotide database; also used for human contaminant screening with a custom human CO1 database)
- **CUTADAPT** (Used with default settings (up to 10% primer mismatch) to retrieve primer-trimmed sequences and trim them to 200 bp fragments simulating Illumina reads prior to BLAST classification)
- **RDP classifier** (Comparator method: RDP classifier v2.12 with the CO1 Eukaryote v1 training set (0.60 bootstrap cutoff at genus rank) benchmarked against the BLAST top-hit method for speed) — https://github.com/terrimporter/CO1Classifier

## Evaluation signals

- Every top hit counted as 'good' must satisfy percent identity >= 95% AND alignment spanning >= 85% of the query length
- The measured RDP/BLAST throughput ratio on a single processor should be approximately 19x, confirming the reported speed benchmark
- Both methods must classify the identical 200 bp query fragment set so sequences-per-minute values are directly comparable
- The output speed_benchmark.csv should contain per-method sequences-per-minute and the computed speed ratio
- Assignment outcomes should be categorized using the defined outcome types (as in Figure S5) so 'good' vs. other outcomes are separable

## Limitations

- A key assumption of the top BLAST hit method is that the query sequence is actually represented in the database; missing congeneric sequences cause false negatives
- The RDP classifier makes taxonomic assignments about 19 times faster than the top BLAST hit method, so BLAST-based assignment may be impractical at very high throughput
- BLAST-based assignments inherit database annotation errors; one mitigation is to summarize assignments to the genus or other more inclusive ranks

## Evidence

- [abstract] the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method: "the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method"
- [results] the query sequence is actually represented in the database: "using the top BLAST hit method, is that the query sequence is actually represented in the database."
- [discussion] the top BLAST hit method against a locally installed copy of the nucleotide database: "the top BLAST hit method against a locally installed copy of the nucleotide database"
- [methods] scoring top hits as 'good' when percent identity >= 95% and the alignment spans >= 85% of the query length: [evidence span withheld — anchor status: not_found]
- [readme] reference sequences formatted for the RDP classifier, BLAST, and SINTAX are also available: "reference sequences formatted for the RDP classifier, BLAST, and SINTAX are also available"
