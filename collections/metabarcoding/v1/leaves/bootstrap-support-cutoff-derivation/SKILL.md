---
name: bootstrap-support-cutoff-derivation
description: Use when deploying the RDP classifier on CO1 metabarcodes and you need
  defensible bootstrap support cutoffs per taxonomic rank and query fragment length
  (500 bp+, 400, 200, 100, 50 bp) before filtering assignments.
when_to_use_negative:
- Input sequences are not expected to be represented in the reference set — the classifier
  is not meant to classify taxa absent from the training database, and leave-one-out
  estimates assume the query exists in the reference
- 'You need species-rank assignment: the genus-trained CO1 Eukaryote v1 set supports
  genus-level cutoffs, but species-rank cutoffs (NA for 99% correct) require the species-trained
  v2 set'
- A cutoff matrix already exists for your exact training-set version and fragment
  lengths (e.g. README tables for v3 or v5.1.0) — reuse it instead of re-deriving
edam_operation: http://edamontology.org/operation_0360
tools:
- name: RDP classifier
  role: Naive Bayesian classifier used for leave-one-out testing, cross-validation
    testing, and classifier training; bootstrap support is estimated by 100 repeated
    re-assignments from randomly subsampled 8 bp words
  repo: https://github.com/terrimporter/CO1Classifier
- name: BioPerl
  role: Perl with BioPerl modules and the Ebot script used to assemble and process
    the CO1 reference sequence sets
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:dbf33c87e0baf51d506b7a5cefcb61f7b4ffa1a9487d8d151a3dda9f8c4addd4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BioPerl
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0360
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bootstrap-support-cutoff-derivation

## Summary

Empirically derives rank- and fragment-length-specific minimum RDP classifier bootstrap support cutoffs that yield at least 99% correct taxonomic assignments, by leave-one-out testing of CO1 reference sets (e.g. CO1 Eukaryote v1) with repeated bootstrap subsampling of 8 bp words.

## When to use

Use when deploying the RDP classifier on CO1 metabarcodes and you need defensible bootstrap support cutoffs per taxonomic rank and query fragment length (500 bp+, 400, 200, 100, 50 bp) before filtering assignments.

## When NOT to use

- Input sequences are not expected to be represented in the reference set — the classifier is not meant to classify taxa absent from the training database, and leave-one-out estimates assume the query exists in the reference
- You need species-rank assignment: the genus-trained CO1 Eukaryote v1 set supports genus-level cutoffs, but species-rank cutoffs (NA for 99% correct) require the species-trained v2 set
- A cutoff matrix already exists for your exact training-set version and fragment lengths (e.g. README tables for v3 or v5.1.0) — reuse it instead of re-deriving

## Inputs

- CO1 training set taxonomy and FASTA files (e.g. CO1 Eukaryote v1 from github.com/terrimporter/CO1Classifier)
- Trained RDP classifier properties file (rRNAClassifier.properties) for the chosen training-set version
- Held-out/reference query sequences trimmed to 500 bp+, 400 bp, 200 bp, 100 bp, and 50 bp fragments
- Known taxonomy assignments for each held-out sequence

## Outputs

- Matrix of minimum bootstrap support cutoffs per taxonomic rank and fragment length achieving 99% correct assignments (table2_bootstrap_cutoffs.csv / paper Table 2)
- Per-query bootstrap support estimates (100 re-assignments from subsampled 8 bp words)

## How to apply

Download a CO1 training set (taxonomy + FASTA, e.g. CO1 Eukaryote v1 from github.com/terrimporter/CO1Classifier) and run leave-one-out testing with RDP classifier v2.12 on full-length 500 bp+ sequences plus 400 bp, 200 bp, 100 bp, and 50 bp fragments, classifying each held-out sequence with a full set of 8 bp 'words' subsampled from the query. For every query, estimate bootstrap support by randomly subsampling the 8 bp words, re-assigning, and repeating 100 times, scoring an assignment as correct when it matches the known taxonomy. Aggregate assignment outcomes across bootstrap support thresholds for each combination of taxonomic rank and fragment length, then identify the minimum cutoff per cell that yields at least 99% correct assignments. In the 2018 paper this produced genus-rank minimum cutoffs of 70% for 500 bp+, 60% for 400 bp, 60% for 200 bp, 60% for 100 bp, and N/A for 50 bp; note that later training-set releases (e.g. v3, v5.1.0) re-derive lower cutoffs via cross-validation, so always recompute against the exact training set version in use. Output the cutoff matrix (analogous to the paper's Table 2) as a CSV for downstream assignment filtering.

## Related tools

- **RDP classifier** (Naive Bayesian classifier used for leave-one-out testing, cross-validation testing, and classifier training; bootstrap support is estimated by 100 repeated re-assignments from randomly subsampled 8 bp words) — https://github.com/terrimporter/CO1Classifier
- **BioPerl** (Perl with BioPerl modules and the Ebot script used to assemble and process the CO1 reference sequence sets) — https://github.com/terrimporter/CO1Classifier

## Examples

```
java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Each reported minimum cutoff must achieve at least 99% correct assignments in the leave-one-out test, matching the paper's Table 2 criterion
- Cutoff values should increase with shorter fragments and more specific ranks; genus rank must be N/A (no achievable cutoff) at 50 bp
- Genus-rank 500 bp+ cutoff must reproduce 70% (paper v1 set), and rows for Superkingdom–Phylum should show cutoff 0 in equivalent tables
- Species-rank row should be NA for the genus-trained set (no cutoff yields 99% correct assignments)
- Bootstrap support per query should be computed from exactly 100 re-assignment replicates over subsampled 8 bp words

## Limitations

- The classifier is not meant to classify taxa not represented in the database; leave-one-out results from singletons were excluded and no bootstrap support cutoff could be computed for singleton genera
- NA cells mean no cutoff achieves the target accuracy (e.g. 50 bp genus, species rank at 99%), so no correct-assignment guarantee is possible for those cells
- Cutoffs are training-set-version specific: the 2018 v1 genus cutoffs (70%/60%) differ from README v3 (40/30/30/40/90) and v5.1.0 (0.3 across) values, which were derived with 5-fold cross-validation instead of leave-one-out
- The method has a high false negative rate in practice, so lenient cutoffs trading accuracy for fewer false negatives may be preferable in some pipelines

## Evidence

- [results] we have calculated a matrix of minimum bootstrap support value cutoffs to obtain 99% correct assignments during leave one out testing (Table 2).: [evidence span withheld — anchor status: not_found]
- [results] Genus 70 60 60 60 N/A: "Genus 70 60 60 60 N/A"
- [methods] used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [methods] These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier: "These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier"
- [readme] Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments:"
