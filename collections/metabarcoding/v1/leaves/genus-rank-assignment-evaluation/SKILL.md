---
name: genus-rank-assignment-evaluation
description: Use when classifying CO1 metabarcode reads (e.g., 200 bp primer-anchored
  Illumina fragments) to genus rank with the RDP classifier and you need to set bootstrap
  cutoffs or compare accuracy/throughput against top BLAST hits.
when_to_use_negative:
- When species-rank resolution is required — the genus-trained CO1 Eukaryote v1 set
  supports genus assignments and the species-trained v2 set is needed instead, with
  species cutoffs (e.g., 0.9 for 500bp+ in v5.1.0) yielding only ~95% correct assignments
- When the query taxon is absent from the reference database — leave-one-out testing
  excluded singletons because the classifier is not meant to classify taxa not represented
  in the database, and false negatives occur when congeneric sequences are missing
- When input is already a classified feature/OTU table rather than raw CO1 sequence
  queries
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3697
tools:
- name: RDP classifier v2.12
  role: Naive Bayesian classifier used for leave-one-out testing, cross-validation
    testing, classifier training, and genus-rank taxonomic assignment of CO1 queries
    with bootstrap support cutoffs
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Top BLAST hit method against a locally installed NCBI nucleotide database,
    used as the speed and accuracy baseline (good hits at >= 95% identity over >=
    85% query length)
- name: CUTADAPT v1.10
  role: Trims primers (default settings, up to 10% mismatch in the primer sequence)
    to generate primer-anchored 200 bp query fragments for benchmarking
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:adb8fdbc72c90c7eb3d6717ec73ca7baf9412e3bc605df7742b81b0d99e4c832
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - CUTADAPT
  - CUTADAPT v1.10
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# genus-rank-assignment-evaluation

## Summary

Evaluates genus-rank taxonomic assignment of CO1 metabarcode sequences using the RDP classifier with the CO1 Eukaryote training sets, applying length- and rank-specific bootstrap support cutoffs and benchmarking accuracy and speed against the top BLAST hit method.

## When to use

Use when classifying CO1 metabarcode reads (e.g., 200 bp primer-anchored Illumina fragments) to genus rank with the RDP classifier and you need to set bootstrap cutoffs or compare accuracy/throughput against top BLAST hits.

## When NOT to use

- When species-rank resolution is required — the genus-trained CO1 Eukaryote v1 set supports genus assignments and the species-trained v2 set is needed instead, with species cutoffs (e.g., 0.9 for 500bp+ in v5.1.0) yielding only ~95% correct assignments
- When the query taxon is absent from the reference database — leave-one-out testing excluded singletons because the classifier is not meant to classify taxa not represented in the database, and false negatives occur when congeneric sequences are missing
- When input is already a classified feature/OTU table rather than raw CO1 sequence queries

## Inputs

- Primer-trimmed CO1 metabarcode query sequences (e.g., 200 bp Illumina fragments as FASTA)
- CO1 Eukaryote v1 (or newer) trained RDP classifier files (rRNAClassifier.properties) from the CO1Classifier repository
- Locally installed copy of the NCBI nucleotide database for the top BLAST hit comparison
- CO1 reference training set taxonomy and FASTA files for leave-one-out/cross-validation testing

## Outputs

- Genus-rank taxonomic assignments with bootstrap support proportions for each query sequence
- Throughput measurements (sequences classified per minute) for RDP and BLAST, and the RDP/BLAST speed ratio (expected ~19x)
- Assignment outcome proportions (correct, incorrect, unclassified) at the genus rank by fragment length

## How to apply

Classify query fragments (e.g., 200 bp CUTADAPT primer-trimmed CO1 reads) with the RDP classifier v2.12 against the CO1 Eukaryote v1 training set downloaded from github.com/terrimporter/CO1Classifier. Score genus-rank assignments as 'good' when the bootstrap proportion is 0.60 or greater for 200 bp fragments, while the 70% cutoff applies to full-length (500 bp+) CO1 sequences per the Table 2 matrix of minimum bootstrap support values for 99% correct assignments. Benchmark speed by recording sequences classified per minute for RDP versus top BLAST hits against a locally installed NCBI nucleotide database, where BLAST hits are 'good' at >= 95% identity spanning >= 85% of the query length. Validate assignment quality with leave-one-out testing of the training set (singletons excluded) and cross-validation, and expect the RDP classifier to be ~19x faster than the top BLAST hit method. Note the README's updated (v5.1.0) cutoffs: genus 0.3 for 500bp+ through 100 bp fragments to reach 99% correct assignments.

## Related tools

- **RDP classifier v2.12** (Naive Bayesian classifier used for leave-one-out testing, cross-validation testing, classifier training, and genus-rank taxonomic assignment of CO1 queries with bootstrap support cutoffs) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Top BLAST hit method against a locally installed NCBI nucleotide database, used as the speed and accuracy baseline (good hits at >= 95% identity over >= 85% query length))
- **CUTADAPT v1.10** (Trims primers (default settings, up to 10% mismatch in the primer sequence) to generate primer-anchored 200 bp query fragments for benchmarking)

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- The measured RDP/BLAST throughput ratio on a single processor should be approximately 19x faster for the RDP classifier (Figure 4)
- Genus-rank 'good' assignments for 200 bp fragments require bootstrap proportion >= 0.60; full-length (500 bp+) sequences require the 70% cutoff per Table 2
- Leave-one-out testing with the minimum cutoffs should yield ~99% correct assignments when the query is present in the reference set (README: genus cutoff 0.3 in v5.1.0)
- BLAST top hits are only counted as 'good' when percent identity >= 95% and alignment spans >= 85% of the query length
- Assignment outcome rates should be reported as proportions of different outcome types as defined in Figure S5

## Limitations

- The classifier cannot classify taxa not represented in the database; false negatives occur when congeneric sequences are missing from the database, of particular concern in quarantine situations
- CO1 reference database coverage is incomplete: only 22% of extant Insecta genera (16,285/72,618) have full-length species-rank CO1 barcodes in GenBank
- Cutoff values differ by classifier version and fragment length: the article's Table 2 gives 60% for 200 bp and 70% for 500bp+ (CO1 Eukaryote v1), while README v5.1.0 recommends a genus cutoff of 0.3 across all lengths for 99% correct assignments
- Cutoffs assume query sequences are present in the reference set; the method has a high false negative rate in practice

## Evidence

- [abstract] the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method: "We show that the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method"
- [results] The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2.: "The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2."
- [methods] used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [discussion] the top BLAST hit method against a locally installed copy of the nucleotide database: "the top BLAST hit method against a locally installed copy of the nucleotide database"
- [readme] using these minimum bootstrap support cutoffs should result in at least 99% correct assignments: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
