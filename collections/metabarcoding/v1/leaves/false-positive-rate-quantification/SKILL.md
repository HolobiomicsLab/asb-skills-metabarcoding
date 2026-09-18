---
name: false-positive-rate-quantification
description: Use when decision support for choosing between top BLAST hit and RDP
  classifier taxonomic assignment of CO1 metabarcodes, or for benchmarking any assignment
  method's false positive rate on sequences with known ground-truth taxa.
when_to_use_negative:
- The goal is to assign taxonomy to unknown environmental sequences rather than to
  benchmark false positive rates on sequences with known ground truth.
- Sequences are full-length (500 bp+) CO1 barcodes — the RDP classifier is ~19x faster
  and Table 2 cutoffs differ (70% bootstrap at genus rank for 500 bp+ rather than
  60% for 200 bp).
- Working with species-rank assignment using the CO1 Eukaryote v2 training set instead
  of genus-rank assignment with v1.
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3697
tools:
- name: RDP classifier
  role: Naive Bayesian classifier used (v2.12 with CO1 Eukaryote v1 training set)
    to assign genus-rank taxonomy with bootstrap support; the lower-FPR method being
    benchmarked.
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: blastn (megablast task, evalue 1e-20) against a locally installed CO1 reference
    nucleotide database provides the top-hit assignments whose false positive rate
    is compared.
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:7427c0fba0268b8f0a74f50a770d10a1dd9b6f6c4c1dee887c087c849f33de6c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# false-positive-rate-quantification

## Summary

Quantifies the false positive rate of taxonomic assignment methods for CO1 metabarcodes by counting high-confidence assignments to known-wrong taxa, comparing the top BLAST hit method against the RDP classifier (CO1 Eukaryote v1 training set), which reduces the false positive rate from nearly 100% to about 34%.

## When to use

Decision support for choosing between top BLAST hit and RDP classifier taxonomic assignment of CO1 metabarcodes, or for benchmarking any assignment method's false positive rate on sequences with known ground-truth taxa.

## When NOT to use

- The goal is to assign taxonomy to unknown environmental sequences rather than to benchmark false positive rates on sequences with known ground truth.
- Sequences are full-length (500 bp+) CO1 barcodes — the RDP classifier is ~19x faster and Table 2 cutoffs differ (70% bootstrap at genus rank for 500 bp+ rather than 60% for 200 bp).
- Working with species-rank assignment using the CO1 Eukaryote v2 training set instead of genus-rank assignment with v1.

## Inputs

- Primer-anchored 200 bp CO1 query sequences in FASTA format
- Trained RDP classifier files (CO1 Eukaryote v1 training set, rRNAClassifier.properties)
- Custom CO1 reference nucleotide database (formatted for BLAST)
- Ground-truth taxon labels for each query sequence (known-wrong vs known-correct assignments)

## Outputs

- RDP classifier output (rdp.output) with genus-rank bootstrap support values
- BLAST results table with pident and qcovs columns
- fpr_comparison.csv: side-by-side false positive rates for top BLAST hit method and RDP classifier

## How to apply

Obtain the trained classifier files (CO1 Eukaryote v1 trained set with rRNAClassifier.properties) from the github.com/terrimporter/CO1Classifier release and the primer-anchored 200 bp query sequences to be classified. Run RDP classifier v2.12 (java -Xmx8g -jar classifier.jar classify -t rRNAClassifier.properties -o rdp.output query.fasta) and record bootstrap support for genus-rank assignments, calling 'good' assignments at a bootstrap proportion of 0.60 or greater for 200 bp fragments per Table 2. In parallel, run blastn (megablast task, evalue 1e-20, outfmt with pident and qcovs) against the custom CO1 reference nucleotide database and apply the 'good' top-BLAST-hit criterion: percent identity >= 95% and alignment spanning >= 85% of the original query length. For each method, count high-confidence assignments that map a sequence to a known-wrong taxon and divide by total high-confidence assignments to compute the false positive rate, then tabulate both FPRs side by side (e.g. fpr_comparison.csv). The rationale is that the top BLAST hit method over-assigns when the query sequence is not actually represented in the database, whereas the RDP classifier with the CO1 Eukaryote v1 training set cuts the false positive rate from nearly 100% to 34%.

## Related tools

- **RDP classifier** (Naive Bayesian classifier used (v2.12 with CO1 Eukaryote v1 training set) to assign genus-rank taxonomy with bootstrap support; the lower-FPR method being benchmarked.) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (blastn (megablast task, evalue 1e-20) against a locally installed CO1 reference nucleotide database provides the top-hit assignments whose false positive rate is compared.)

## Examples

```
java -Xmx8g -jar classifier.jar classify -t rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- The reproduced comparison table should show the RDP classifier reduces the false positive rate from nearly 100% (top BLAST hit method) to about 34%.
- Every high-confidence RDP assignment counted toward the FPR must have bootstrap support >= 0.60 at the genus rank for 200 bp fragments.
- Every high-confidence top-BLAST-hit assignment counted must satisfy percent identity >= 95% and alignment spanning >= 85% of the original query length.
- RDP output columns must be correctly aligned (later releases fixed comma-in-field misalignment of columns), and counts must satisfy FPR = (assignments to known-wrong taxa) / (total high-confidence assignments).

## Limitations

- Table 2 (definitions of 'good' assignments for 200 bp fragments) is referenced but may not be included in the provided section; consult the supplementary material for exact cutoff definitions.
- The classifier cannot classify taxa not represented in the reference database; false negatives occur when congeneric sequences are missing from the database.
- Figure S5 (definitions of taxonomic assignment outcome types) is referenced but not included in this section, so outcome-type classification must be reconstructed from the supplementary files.
- The false positive rate depends on database currency — CO1 reference coverage is incomplete (e.g. only 22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank).

## Evidence

- [abstract] RDP classifier reduces the false positive rate from nearly 100% to 34%: "reduce the false positive rate from nearly 100% to 34%"
- [methods] 'Good' assignments for the RDP classifier were defined according to Table 2 for 200 bp fragments at the genus rank: [evidence span withheld — anchor status: not_found]
- [results] 60% bootstrap support cutoff for primer-anchored 200 bp sequences at genus rank: [evidence span withheld — anchor status: not_found]
- [discussion] RDP classifier 2.12 with the CO1 Eukaryote v1 training set is the benchmarked configuration: "the RDP classifier 2.12 with the CO1 Eukaryote v1 training set"
- [readme] README provides the concrete RDP classifier invocation used with the trained CO1 set: "java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta"
