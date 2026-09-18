---
name: leave-one-out-classifier-validation
description: Use when a Naive Bayes/RDP-style metabarcode classifier needs rank- and
  fragment-length-specific bootstrap support cutoffs, or when the minimum confidence
  threshold guaranteeing ~99% correct CO1 assignments must be established before applying
  it to metabarcode queries.
when_to_use_negative:
- Queries may contain taxa not represented in the reference training set — the classifier
  is not meant to classify unrepresented taxa, so leave-one-out cutoffs would overstate
  real-world accuracy
- Species-rank assignments with the genus-trained CO1 Eukaryote v1 set — Table 2 reports
  N/A (no cutoff achieves 99% correct assignments at species rank)
- A different training-set version (e.g. v5.1.0) with its own 5-fold cross-validation
  cutoffs is already published — reusing these v1 leave-one-out cutoffs would mix
  incompatible thresholds (v3 README lists 40% genus at 500 bp+, v5.1.0 lists 0.3)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3050
tools:
- name: RDP classifier v2.12
  role: Naive Bayesian classifier used for leave-one-out testing, cross-validation,
    and classifier training on the CO1 training set
  repo: https://github.com/terrimporter/CO1Classifier
- name: CO1 Eukaryote v1 training set
  role: Genus-rank reference training set (taxonomy + FASTA) whose held-out sequences
    are classified during leave-one-out validation
  repo: https://github.com/terrimporter/CO1Classifier/releases
merged_aliases:
- leave-one-out-cross-validation-testing
merged_alias_records:
- alias: leave-one-out-cross-validation-testing
  slug: leave-one-out-cross-validation-testing
  cosine_score: 0.8306
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:15693e4de04915de8244725f410cd57941d611442296a01237c6f006972df956
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - CO1 Eukaryote v1 training set
  - RDP classifier
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3050
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# Leave-one-out classifier validation

## Summary

Empirically calibrates taxonomic-assignment confidence cutoffs by running RDP classifier v2.12 leave-one-out tests over a CO1 reference training set (e.g. CO1 Eukaryote v1) across fragment lengths, then deriving the minimum bootstrap support cutoff per taxonomic rank that yields at least 99% correct assignments (Table 2).

## When to use

Use when a Naive Bayes/RDP-style metabarcode classifier needs rank- and fragment-length-specific bootstrap support cutoffs, or when the minimum confidence threshold guaranteeing ~99% correct CO1 assignments must be established before applying it to metabarcode queries.

## When NOT to use

- Queries may contain taxa not represented in the reference training set — the classifier is not meant to classify unrepresented taxa, so leave-one-out cutoffs would overstate real-world accuracy
- Species-rank assignments with the genus-trained CO1 Eukaryote v1 set — Table 2 reports N/A (no cutoff achieves 99% correct assignments at species rank)
- A different training-set version (e.g. v5.1.0) with its own 5-fold cross-validation cutoffs is already published — reusing these v1 leave-one-out cutoffs would mix incompatible thresholds (v3 README lists 40% genus at 500 bp+, v5.1.0 lists 0.3)

## Inputs

- CO1 Eukaryote v1 training set: taxonomy file and FASTA reference file (from github.com/terrimporter/CO1Classifier)
- RDP classifier v2.12 (Java classifier.jar)
- Known taxonomic lineage for each reference sequence (ground-truth labels)

## Outputs

- Matrix of minimum bootstrap support cutoffs per taxonomic rank × fragment length yielding ≥99% correct assignments (paper Table 2; e.g. table2_bootstrap_cutoffs.csv)
- Per-rank, per-fragment-length proportions of correct assignments (e.g. accuracy_by_fragment_length.csv)

## How to apply

Download the CO1 Eukaryote v1 training set (taxonomy and FASTA files) from the terrimporter/CO1Classifier release, exclude singleton taxa since the classifier is not meant to classify taxa absent from the database, and run leave-one-out testing with RDP classifier v2.12 for full-length (500 bp+) sequences and 400, 200, 100, and 50 bp fragments. For each held-out sequence, estimate bootstrap support by randomly subsampling the full set of 8 bp words from the query, re-classifying, and repeating 100 times. Score each assignment as correct only when it matches the known taxonomy, then aggregate outcomes across bootstrap support thresholds for each rank × fragment-length combination. Report the minimum cutoff that achieves ≥99% correct assignments; for the v1 genus-rank training this gives 70% (500 bp+), 60% (400 bp), 60% (200 bp), 60% (100 bp), and N/A (50 bp). Apply the selected cutoff (e.g. 70% for full-length genus-rank calls) when classifying real metabarcode queries.

## Related tools

- **RDP classifier v2.12** (Naive Bayesian classifier used for leave-one-out testing, cross-validation, and classifier training on the CO1 training set) — https://github.com/terrimporter/CO1Classifier
- **CO1 Eukaryote v1 training set** (Genus-rank reference training set (taxonomy + FASTA) whose held-out sequences are classified during leave-one-out validation) — https://github.com/terrimporter/CO1Classifier/releases

## Examples

```
java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Every reported minimum cutoff, when applied to its rank × fragment-length leave-one-out results, yields at least 99% correct assignments (Table 2 criterion)
- Singleton taxa are excluded from the leave-one-out tally because assignments for taxa absent from the database are meaningless
- Cutoffs increase with shorter fragments and more specific ranks (e.g. genus: 70% at 500 bp+ down to N/A at 50 bp); a decreasing pattern signals a pipeline error
- Assignment is scored correct only when it matches the known taxonomy of the held-out sequence, not merely an assignment being returned
- Bootstrap support per query derives from ~100 repeated classifications on randomly subsampled 8 bp words, not a single classification run

## Limitations

- No cutoff can reach 99% correct assignments at species rank with the genus-trained set (N/A in Table 2); the method has a high false negative rate in practice, and the README offers 95%/90% alternative cutoff tables
- Cutoffs assume query sequences are present in the reference set; database coverage gaps (only 22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank) cause false negatives, of particular concern in quarantine situations
- Cutoff values are training-set-version specific: paper v1 values (70% genus at 500 bp+) differ from v3 (40%) and v5.1.0 (0.3) values released in the repository

## Evidence

- [results] we have calculated a matrix of minimum bootstrap support value cutoffs to obtain 99% correct assignments during leave one out testing (Table 2).: [evidence span withheld — anchor status: not_found]
- [results] Since the classifier is not meant to classify taxa not represented in the database, leave-one-out testing results from singletons were excluded from this figure: "Since the classifier is not meant to classify taxa not represented in the database, leave-one-out testing results from singletons were excluded from this figure and no bootstrap support cutoff was"
- [results] Classifier accuracy is highest at more inclusive taxonomic ranks, especially for fragments 200 bp or longer.: [evidence span withheld — anchor status: not_found]
- [readme] Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments:: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments:"
- [readme] NA = No cutoff available will result in 99% correct assignments: "NA = No cutoff available will result in 99% correct assignments"
