---
name: blast-percent-identity-assignment-criteria
description: Use when assigning taxonomy to CO1 metabarcode (primer-anchored ~200
  bp or longer) query sequences via a top BLAST hit, or when benchmarking a classifier's
  false positive rate against the popular top BLAST hit approach.
when_to_use_negative:
- When queries are likely absent from the reference database — the method implicitly
  assumes the query sequence is represented, and absent taxa produce confident misassignments
  (near-100% FPR)
- When a lower false positive rate matters more than speed — the RDP classifier with
  the CO1 Eukaryote v1 training set and a 0.60 genus-rank bootstrap cutoff reduces
  FPR to ~34% and is ~19x faster
- When assigning at the species rank with the current CO1Classifier releases — README
  cutoff tables provide no usable species-rank cutoff achieving 99% correct assignments
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: BLAST
  role: blastn (megablast task, evalue 1e-20) searched against a locally installed
    custom CO1 nucleotide database to obtain pident and qcovs used for top-hit assignment
    filtering
- name: RDP classifier 2.12
  role: comparator method whose assignments (CO1 Eukaryote v1 training set, 0.60 genus-rank
    bootstrap cutoff) are used to benchmark the top BLAST hit false positive rate
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:25fad99e3b86011d3e9c94d3736caa59c0600e8ce927adabd40cdfe986a54b6b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - RDP classifier 2.12
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# blast-percent-identity-assignment-criteria

## Summary

Applies the paper's 'good' top-BLAST-hit taxonomic assignment criterion for CO1 metabarcodes: run blastn (megablast task, evalue 1e-20) against a CO1 reference nucleotide database and accept an assignment only when percent identity >= 95% and alignment covers >= 85% of the query length. It is the baseline method the RDP CO1 classifier was benchmarked against.

## When to use

Use when assigning taxonomy to CO1 metabarcode (primer-anchored ~200 bp or longer) query sequences via a top BLAST hit, or when benchmarking a classifier's false positive rate against the popular top BLAST hit approach.

## When NOT to use

- When queries are likely absent from the reference database — the method implicitly assumes the query sequence is represented, and absent taxa produce confident misassignments (near-100% FPR)
- When a lower false positive rate matters more than speed — the RDP classifier with the CO1 Eukaryote v1 training set and a 0.60 genus-rank bootstrap cutoff reduces FPR to ~34% and is ~19x faster
- When assigning at the species rank with the current CO1Classifier releases — README cutoff tables provide no usable species-rank cutoff achieving 99% correct assignments

## Inputs

- CO1 query sequences in FASTA format (e.g., primer-anchored 200 bp metabarcodes)
- CO1 reference nucleotide database (BLAST-format reference sequences from the CO1Classifier release)
- Known/expected taxonomy labels for benchmark queries (to compute false positive rate)

## Outputs

- blastn tabular output with qseqid, sseqid, pident, qcovs per hit
- High-confidence taxonomic assignments per query passing the 95% identity / 85% coverage criterion
- False positive rate table comparing top BLAST hit vs RDP classifier (e.g., fpr_comparison.csv)

## How to apply

Build or obtain a custom CO1 reference nucleotide database from the CO1Classifier repository release, then run blastn with the megablast task and evalue 1e-20, requesting pident and qcovs in the outfmt. Apply the 'good' top-BLAST-hit criterion from Porter & Hajibabaei (2018): a hit is accepted only if percent identity >= 95% and the alignment spans >= 85% of the original query length; the assignment is the taxonomy of the single top qualifying hit. The rationale is that the method implicitly assumes the query sequence is actually represented in the database — metagenomic CO1 queries that are not represented yield confidently wrong assignments, which is why this method showed a false positive rate of nearly 100% versus 34% for the RDP classifier with the CO1 Eukaryote v1 training set at a 0.60 genus-rank bootstrap cutoff. Tabulate assignments per query (e.g., fpr_comparison.csv) and, when benchmarking, count high-confidence assignments to known-wrong taxa divided by total high-confidence assignments to compute the false positive rate.

## Related tools

- **BLAST** (blastn (megablast task, evalue 1e-20) searched against a locally installed custom CO1 nucleotide database to obtain pident and qcovs used for top-hit assignment filtering)
- **RDP classifier 2.12** (comparator method whose assignments (CO1 Eukaryote v1 training set, 0.60 genus-rank bootstrap cutoff) are used to benchmark the top BLAST hit false positive rate) — https://github.com/terrimporter/CO1Classifier

## Examples

```
blastn -task megablast -evalue 1e-20 -outfmt '6 qseqid sseqid pident qcovs' -query query.fasta -db co1_ref_nucleotide > blast.output
```

## Evaluation signals

- Every accepted assignment has pident >= 95% AND qcovs >= 85% of the original query length in the blastn output
- blastn was run with the megablast task and evalue 1e-20 against the CO1 reference database
- Benchmark reproduction yields a false positive rate near 100% for top BLAST hit vs ~34% for the RDP classifier (CO1 Eukaryote v1, 0.60 genus bootstrap cutoff)
- Reproduced comparison table (fpr_comparison.csv) lists FPR values for both methods side by side for the same query set

## Limitations

- The top BLAST hit method has a false positive rate of nearly 100% when query sequences are not represented in the database
- The method only succeeds when the query sequence is actually present in the database; congenerically missing taxa cause false negatives of concern in quarantine situations
- GenBank annotation errors propagate into assignments; the article recommends summarizing to genus or more inclusive ranks to mitigate this

## Evidence

- [methods] Run blastn against the custom CO1 reference nucleotide database (megablast task, evalue 1e-20, outfmt with pident and qcovs): [evidence span withheld — anchor status: not_found]
- [discussion] the top BLAST hit method against a locally installed copy of the nucleotide database: "the top BLAST hit method against a locally installed copy of the nucleotide database"
- [results] using the top BLAST hit method, is that the query sequence is actually represented in the database.: "using the top BLAST hit method, is that the query sequence is actually represented in the database."
- [readme] As of version 5.0.0, reference sequences formatted for the RDP classifier, BLAST, and SINTAX are also available.: "reference sequences formatted for the RDP classifier, BLAST, and SINTAX are also available"
