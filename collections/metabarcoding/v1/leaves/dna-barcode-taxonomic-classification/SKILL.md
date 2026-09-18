---
name: dna-barcode-taxonomic-classification
description: Use when assigning taxonomy to CO1 amplicon/metabarcode sequences (500
  bp to 50 bp fragments) with the RDP classifier, or when selecting minimum bootstrap
  support cutoffs per taxonomic rank and sequence length.
when_to_use_negative:
- Query taxa are expected to be absent from the reference training set — the classifier
  cannot classify taxa not represented in the database and congeneric-sequence gaps
  cause false negatives, a particular concern in quarantine situations
- Species-rank assignment is required — the genus-ranked training sets (e.g., v1)
  report 'NA' cutoffs at species rank; use the species-ranked set (v2) or the species-level
  95%-accuracy table instead
- Target marker is not CO1 — for prokaryote 16S or fungal ITS use SILVA, Greengenes,
  or UNITE reference sets rather than this CO1 training set
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: RDP classifier v2.12
  role: Naive Bayesian classifier used to make genus-rank CO1 taxonomic assignments
    with bootstrap support values; also used for leave-one-out testing, cross-validation
    testing, and classifier training
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Top-hit comparison method benchmarked against the RDP classifier (queries
    per minute), with 'good' hits defined as >=95% identity spanning >=85% of the
    query length
- name: CUTADAPT v1.10
  role: Retrieves primer-trimmed CO1 sequences (up to 10% primer mismatch) trimmed
    to 200 bp to simulate Illumina reads for the classification speed benchmark
merged_aliases:
- dna-barcode-taxonomic-assignment-benchmarking
merged_alias_records:
- alias: dna-barcode-taxonomic-assignment-benchmarking
  slug: dna-barcode-taxonomic-assignment-benchmarking
  cosine_score: 0.8066
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:bce35d79f2c1a1027a18f322ab9aa4ca2da600a15d2828397767f73d4c75e85a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - CUTADAPT
  - CUTADAPT v1.10
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# dna-barcode-taxonomic-classification

## Summary

Taxonomically classify animal CO1 metabarcode sequences using the RDP Naive Bayesian classifier with the CO1 Eukaryote training sets from the CO1Classifier repository, applying rank- and fragment-length-specific bootstrap support cutoffs to obtain at least 99% correct assignments.

## When to use

Use when assigning taxonomy to CO1 amplicon/metabarcode sequences (500 bp to 50 bp fragments) with the RDP classifier, or when selecting minimum bootstrap support cutoffs per taxonomic rank and sequence length.

## When NOT to use

- Query taxa are expected to be absent from the reference training set — the classifier cannot classify taxa not represented in the database and congeneric-sequence gaps cause false negatives, a particular concern in quarantine situations
- Species-rank assignment is required — the genus-ranked training sets (e.g., v1) report 'NA' cutoffs at species rank; use the species-ranked set (v2) or the species-level 95%-accuracy table instead
- Target marker is not CO1 — for prokaryote 16S or fungal ITS use SILVA, Greengenes, or UNITE reference sets rather than this CO1 training set

## Inputs

- CO1 query sequences in FASTA format (full-length 500 bp+ or metabarcode fragments of 400/300/200/100/50 bp)
- Trained RDP classifier files (rRNAClassifier.properties) from a CO1Classifier release (e.g., RDP_COIv5.1.0.zip)
- Original taxonomy and FASTA reference files ('version-ref') if retraining the classifier

## Outputs

- RDP classifier output assigning each query to a taxonomic lineage with genus-rank bootstrap support values
- List of 'good' assignments filtered at the rank-specific bootstrap cutoff (e.g., 70% genus cutoff for 500 bp+ with v1)
- Optional speed/accuracy comparison table vs. top BLAST hit (e.g., speed_benchmark.csv)

## How to apply

Download a trained CO1 classifier release from github.com/terrimporter/CO1Classifier and run the RDP classifier (v2.12 in the paper; v2.13 installable via conda per the README) against the rRNAClassifier.properties file of the trained set, classifying queries to the genus rank (v1/v2 sets support genus- and species-rank assignments respectively). Score a genus assignment as 'good' only when the bootstrap proportion meets the rank-specific cutoff — for the paper's CO1 Eukaryote v1 set these are 70% for 500 bp+ sequences and 60% for 400, 200, and 100 bp fragments at genus rank, values validated by leave-one-out testing to yield 99% correct assignments. Cutoffs are version-dependent: the README's v3 set recommends lower cutoffs (e.g., genus 40 for 500 bp+) and the v5.1.0 cross-validated set uses 0.3 at genus rank across all lengths, so always use the cutoff table published with the release you trained on. When benchmarking against BLAST, trim queries (e.g., CUTADAPT with up to 10% primer mismatch, 200 bp fragments) and compare throughput; the paper reports the RDP classifier is ~19x faster than the top BLAST hit method (>=95% identity, >=85% query coverage). Assume query sequences are represented in the reference set — the classifier is not meant to classify taxa absent from the database.

## Related tools

- **RDP classifier v2.12** (Naive Bayesian classifier used to make genus-rank CO1 taxonomic assignments with bootstrap support values; also used for leave-one-out testing, cross-validation testing, and classifier training) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Top-hit comparison method benchmarked against the RDP classifier (queries per minute), with 'good' hits defined as >=95% identity spanning >=85% of the query length)
- **CUTADAPT v1.10** (Retrieves primer-trimmed CO1 sequences (up to 10% primer mismatch) trimmed to 200 bp to simulate Illumina reads for the classification speed benchmark)

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Genus-rank assignments meeting the Table 2 cutoff (70% for 500 bp+, 60% for 400/200/100 bp fragments with CO1 Eukaryote v1) should achieve at least 99% correct assignments in leave-one-out testing
- Speed benchmark should reproduce a roughly 19x throughput ratio of RDP classifier over the top BLAST hit method on a single processor
- Every classified query in the RDP output must carry a bootstrap support value that can be compared against the release-specific cutoff table (v3 vs v5.1.0 cutoffs differ substantially)
- Queries with bootstrap support below the cutoff must be reported as unassigned ('good' assignments only at or above the threshold, e.g., genus rank >= 0.60 for 200 bp fragments)

## Limitations

- Cutoffs assume query sequences are present in the reference set; the method has a high false negative rate in practice, which is why the README also provides more lenient ~90% and species-level ~95% cutoff tables
- Bootstrap cutoffs are release-specific and change with training-set updates (paper v1: genus 70% at 500 bp+; README v3: 40; v5.1.0 cross-validated: 0.3), so values must not be mixed across classifier versions
- CO1 reference database coverage is incomplete (e.g., only 22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank), and there is no curated community-maintained CO1 database equivalent to SILVA, Greengenes, or UNITE

## Evidence

- [results] we have calculated a matrix of minimum bootstrap support value cutoffs to obtain 99% correct assignments during leave one out testing (Table 2): [evidence span withheld — anchor status: not_found]
- [abstract] the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method: "the RDP classifier can make taxonomic assignments about 19 times faster than the popular top BLAST hit method"
- [methods] used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [readme] The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file): "The trained files ready to be used with the RDP Classifier are available as well as the original files used for training (a taxonomy file and a FASTA file)"
- [readme] Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
