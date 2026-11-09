---
name: naive-bayesian-taxonomic-classification
description: Use when assigning taxonomy to CO1 mtDNA metabarcode reads or full-length
  barcode sequences against a curated CO1 reference training set, and when a bootstrap-support
  cutoff is needed to control assignment accuracy.
when_to_use_negative:
- When query taxa are not represented in the training set — the classifier cannot
  meaningfully assign taxa absent from the database and false negatives occur when
  congeneric sequences are missing
- When the input is already a taxonomic feature table or BLAST hit list rather than
  raw FASTA query sequences
- When a curated community-maintained CO1 equivalent of SILVA/Greengenes/UNITE with
  third-party annotation is required — none exists; the user community maintains CO1
  reference data
edam_operation: http://edamontology.org/operation_0363
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: RDP classifier
  role: Naive Bayesian classifier used to make taxonomic assignments to CO1 queries
    using a CO1-specific training set, with bootstrap support as assignment confidence
  repo: https://github.com/terrimporter/CO1Classifier
- name: RDP classifier v2.12
  role: Version used in the paper for leave-one-out testing, cross-validation testing,
    and classifier training of the CO1 training sets
  repo: https://github.com/terrimporter/CO1Classifier/releases
- name: BLAST
  role: Top BLAST hit method used as the comparison baseline for speed and false-positive
    rate of the RDP classifier assignments
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:944228ba445e495302343224f183d0ccca08ee7a6e973b2e988db8cb7a64e2aa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - RDP classifier
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_0363
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# naive-bayesian-taxonomic-classification

## Summary

Use the RDP naive Bayesian classifier with a curated CO1 training set (e.g., CO1 Eukaryote v1 from the CO1Classifier repo) to assign taxonomy to CO1 metabarcode sequences, with bootstrap support as an assignment-confidence measure. It is far faster than top BLAST hit matching (~19x) and reduces false positives.

## When to use

Use when assigning taxonomy to CO1 mtDNA metabarcode reads or full-length barcode sequences against a curated CO1 reference training set, and when a bootstrap-support cutoff is needed to control assignment accuracy.

## When NOT to use

- When query taxa are not represented in the training set — the classifier cannot meaningfully assign taxa absent from the database and false negatives occur when congeneric sequences are missing
- When the input is already a taxonomic feature table or BLAST hit list rather than raw FASTA query sequences
- When a curated community-maintained CO1 equivalent of SILVA/Greengenes/UNITE with third-party annotation is required — none exists; the user community maintains CO1 reference data

## Inputs

- Query CO1 sequences in FASTA format (full-length 500 bp+ or truncated fragments 400/200/100/50 bp)
- RDP-formatted CO1 training set (taxonomy file + FASTA reference file, e.g., CO1 Eukaryote v1/v2 or the v5.1.0 release RDP_COIv5.1.0.zip)
- Trained classifier properties file (rRNAClassifier.properties)

## Outputs

- Per-sequence taxonomic assignments at each rank with bootstrap support values (rdp.output)
- Leave-one-out accuracy tallies per taxonomic rank per fragment length (e.g., accuracy_by_fragment_length.csv)
- Matrix of minimum bootstrap support cutoffs for 99% correct assignments (Table 2 analogue)

## How to apply

Train or download a pre-trained CO1 training set (taxonomy + FASTA files from the CO1Classifier GitHub release) and run the RDP classifier (v2.12 in the paper) in classify mode with the trained rRNAClassifier.properties file. Exclude singleton taxa when benchmarking, since assignments are only meaningful for taxa represented in the database. For performance assessment, run leave-one-out testing (each sequence removed from training, classified using 8 bp words subsampled from the query, then returned) and repeat classification 100 times with random 8 bp word subsamples to record the proportion of times the original assignment is recovered (bootstrap support). Apply the rank- and fragment-length-specific minimum bootstrap cutoffs (e.g., 70% genus for 500 bp+ sequences; 60% genus for 200 bp sequences) from the published Table 2 to reach ~99% correct assignments. Score assignments as correct when they match the known taxonomy of the query and tally per taxonomic rank per fragment length.

## Related tools

- **RDP classifier** (Naive Bayesian classifier used to make taxonomic assignments to CO1 queries using a CO1-specific training set, with bootstrap support as assignment confidence) — https://github.com/terrimporter/CO1Classifier
- **RDP classifier v2.12** (Version used in the paper for leave-one-out testing, cross-validation testing, and classifier training of the CO1 training sets) — https://github.com/terrimporter/CO1Classifier/releases
- **BLAST** (Top BLAST hit method used as the comparison baseline for speed and false-positive rate of the RDP classifier assignments)

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Assignment counts as correct when the assignment matches the known taxonomy of the query; per-rank per-fragment-length proportion correct should be tabulated
- Leave-one-out (or 5-fold cross-validation in v5.1.0) accuracy should exceed 99% at ranks where the recommended minimum bootstrap cutoff is applied
- Classifier accuracy should be highest at more inclusive taxonomic ranks, especially for fragments 200 bp or longer; performance should be similar across CO1 fragment lengths
- Genus-rank assignments of 500 bp+ queries require a 70% bootstrap support cutoff; 200 bp primer-anchored queries require 60%
- Speed check: RDP assignments should be roughly 19 times faster than the top BLAST hit method on the same query set

## Limitations

- No cutoff achieves 99% correct species-rank assignments (NA cells in the cutoff tables); species-level work needs the 95% cutoff table (e.g., species 60% for 500 bp+ in v3)
- Assignments are meaningless for taxa absent from the training set; singleton taxa were excluded from leave-one-out benchmarking
- Reference database coverage gaps remain: only 22% of extant Insecta genera (16,285/72,618) are represented by full-length species-rank CO1 barcodes in GenBank
- The method shows a high false negative rate in practice, so lenient cutoffs (90% correct assignments) are offered as an alternative

## Evidence

- [abstract] RDP classifier speed vs BLAST: "the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method"
- [results] fragment length accuracy: [evidence span withheld — anchor status: not_found]
- [results] bootstrap cutoff matrix: [evidence span withheld — anchor status: not_found]
- [methods] RDP version and use: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [readme] README cutoff claim: "using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
