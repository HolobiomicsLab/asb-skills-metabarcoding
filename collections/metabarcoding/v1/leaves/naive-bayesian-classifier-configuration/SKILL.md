---
name: naive-bayesian-classifier-configuration
description: Use when assigning CO1 amplicon or full-length barcode queries to genus
  rank with the RDP classifier and a rank-specific bootstrap support threshold must
  be chosen for a given fragment length (500 bp+ down to 50 bp).
when_to_use_negative:
- The goal is species-rank assignment with the genus-trained CO1 Eukaryote v1 set
  — no bootstrap cutoff achieves 99% correct species assignments (Species = NA in
  Table 2); use the species-trained v2 set instead
- Queries are expected to be absent from the reference set — the classifier is not
  meant to classify taxa not represented in the database, so leave-one-out-derived
  cutoffs and their accuracy guarantees do not apply
- A similarity-based workflow is required instead, e.g. when the query sequence must
  literally be present in the database (top BLAST hit method)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: RDP classifier
  role: Naïve Bayesian classifier used with the CO1 Eukaryote v1 training set; version
    2.12 used for leave-one-out testing, cross-validation testing, and classifier
    training; bootstrap support estimated via 100-fold random subsampling of 8 bp
    words
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: 'Comparison baseline: top BLAST hit method against the local nucleotide database,
    ~19x slower than the RDP classifier; also used to screen human contaminants in
    training-set construction'
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:1aa0ae9e9d671235c256fdb83700bce2ccf80e0a6c6ad3c7f41356c1680e553b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - RDP classifier
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# naive-bayesian-classifier-configuration

## Summary

Configure the RDP naïve Bayesian classifier for CO1 metabarcode taxonomic assignment by calibrating rank- and fragment-length-specific bootstrap support cutoffs so that at least 99% of assignments are correct during leave-one-out or cross-validation testing.

## When to use

Use when assigning CO1 amplicon or full-length barcode queries to genus rank with the RDP classifier and a rank-specific bootstrap support threshold must be chosen for a given fragment length (500 bp+ down to 50 bp).

## When NOT to use

- The goal is species-rank assignment with the genus-trained CO1 Eukaryote v1 set — no bootstrap cutoff achieves 99% correct species assignments (Species = NA in Table 2); use the species-trained v2 set instead
- Queries are expected to be absent from the reference set — the classifier is not meant to classify taxa not represented in the database, so leave-one-out-derived cutoffs and their accuracy guarantees do not apply
- A similarity-based workflow is required instead, e.g. when the query sequence must literally be present in the database (top BLAST hit method)

## Inputs

- CO1 Eukaryote training set (taxonomy file + FASTA reference file, e.g. CO1 Eukaryote v1) from github.com/terrimporter/CO1Classifier
- CO1 query sequences in FASTA format (full-length 500 bp+ barcodes or 400/200/100/50 bp fragments)
- Known taxonomy labels for training sequences (used to score correct vs incorrect assignments)

## Outputs

- Matrix of minimum bootstrap support cutoffs per taxonomic rank and fragment length achieving >=99% correct assignments (e.g. table2_bootstrap_cutoffs.csv)
- RDP classifier output file with bootstrap support values per query assignment (e.g. rdp.output)

## How to apply

Download the CO1 Eukaryote training set (taxonomy + FASTA files) from github.com/terrimporter/CO1Classifier and use it directly with RDP classifier v2.12 without additional user training. Perform leave-one-out testing on the training set (singletons excluded) for full-length 500 bp+ sequences and 400 bp, 200 bp, 100 bp, and 50 bp fragments, classifying each removed sequence using a full set of 8 bp 'words' subsampled from the query. For every query, estimate bootstrap support by randomly subsampling the 8 bp words, re-assigning, and repeating 100 times, scoring an assignment as correct when it matches the known taxonomy. Aggregate outcomes across bootstrap thresholds per taxonomic rank and fragment length, then select the minimum cutoff per rank/length that yields at least 99% correct assignments — for the 2018 CO1 Eukaryote v1 set this gives genus-rank cutoffs of 70% (500 bp+), 60% (400 bp), 60% (200 bp), 60% (100 bp), and N/A (50 bp). Apply the selected cutoff (e.g. 70% for full-length genus-rank assignments) when classifying real metabarcode reads; later README versions (v3, v5.1.0) report different, version-specific cutoff matrices, so cutoffs must match the training-set release in use.

## Related tools

- **RDP classifier** (Naïve Bayesian classifier used with the CO1 Eukaryote v1 training set; version 2.12 used for leave-one-out testing, cross-validation testing, and classifier training; bootstrap support estimated via 100-fold random subsampling of 8 bp words) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Comparison baseline: top BLAST hit method against the local nucleotide database, ~19x slower than the RDP classifier; also used to screen human contaminants in training-set construction)

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Each reported minimum cutoff must achieve at least 99% correct assignments in leave-one-out testing, matching the paper's Table 2 criterion
- Genus-rank cutoffs for the 2018 v1 training set must reproduce the paper's values: 70% (500 bp+), 60% (400 bp), 60% (200 bp), 60% (100 bp), N/A (50 bp)
- Bootstrap support for each query is estimated by 100 independent random subsamplings of the 8 bp words — verify the repetition count is 100
- The applied cutoff matches the training-set release: README v3 reports genus cutoffs of 40/30/30/40/90 and v5.1.0 reports 0.3 across lengths, so a 70% cutoff is only valid for the v1-era testing described in the paper
- Classification performance should remain similar across the length of the CO1 barcoding region, with accuracy highest at more inclusive taxonomic ranks especially for fragments >=200 bp

## Limitations

- Cutoffs are training-set-version specific: the paper's Table 2 values (genus 70% for 500 bp+) differ from the v3 README values (genus 40%) and the v5.1.0 values (genus 0.3), which were derived from different reference sets and 5-fold cross-validation
- Species-rank assignment with the genus-trained classifier has no cutoff achieving 99% correct assignments (Species = NA); a high false negative rate in practice is noted, and README v3.0+ offers more lenient cutoffs targeting ~90% correct assignments
- The classifier cannot assign taxa absent from the database — false negatives occur when congeneric sequences are missing, a particular concern in quarantine situations

## Evidence

- [results] we have calculated a matrix of minimum bootstrap support value cutoffs to obtain 99% correct assignments during leave one out testing (Table 2).: [evidence span withheld — anchor status: not_found]
- [results] The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2.: "The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2."
- [methods] used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [readme] Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
- [results] Since the classifier is not meant to classify taxa not represented in the database, leave-one-out testing results from singletons were excluded from this figure: "Since the classifier is not meant to classify taxa not represented in the database, leave-one-out testing results from singletons were excluded from this figure"
