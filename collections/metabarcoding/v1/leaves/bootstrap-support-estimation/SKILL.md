---
name: bootstrap-support-estimation
description: Use when you need bootstrap support values or minimum cutoff thresholds
  (e.g. 70% for genus, 500 bp+ CO1) for RDP classifier taxonomic assignments of CO1
  sequences, or when validating classifier accuracy across fragment lengths (50–500+
  bp) and taxonomic ranks.
when_to_use_negative:
- Query taxa are not represented in the training database (e.g. singleton taxa excluded
  from leave-one-out testing) — classifier assignments and their bootstrap support
  are not meaningful
- Working with rDNA metabarcode markers (16S, ITS) whose accuracy shows distinctive
  stem-loop variation rather than the length-insensitive behavior of CO1
- You need species-rank assignments — no bootstrap cutoff yields 99% correct species
  assignments (NA in Table 2); fall back to genus or more inclusive ranks
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: RDP classifier
  role: Naïve Bayesian classifier run with the CO1 Eukaryote v1 training set for leave-one-out
    testing and 100× bootstrap re-classification from subsampled 8 bp words
  repo: https://github.com/terrimporter/CO1Classifier
- name: CO1 Eukaryote v1 training set
  role: Reference taxonomy and FASTA files used to train the RDP classifier and derive
    bootstrap cutoffs
  repo: https://github.com/terrimporter/CO1Classifier/releases
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:1942f931bdd4ceaa189747e032cabeb7df756f2bfb2e1af3fb925bb75be85a8c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - CO1 Eukaryote v1 training set
  - RDP classifier
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# bootstrap-support-estimation

## Summary

Estimates confidence in RDP classifier taxonomic assignments of CO1 metabarcode sequences by randomly subsampling 8 bp words from each query and repeating classification 100 times, then deriving minimum bootstrap support cutoffs per rank and fragment length via leave-one-out testing.

## When to use

Use when you need bootstrap support values or minimum cutoff thresholds (e.g. 70% for genus, 500 bp+ CO1) for RDP classifier taxonomic assignments of CO1 sequences, or when validating classifier accuracy across fragment lengths (50–500+ bp) and taxonomic ranks.

## When NOT to use

- Query taxa are not represented in the training database (e.g. singleton taxa excluded from leave-one-out testing) — classifier assignments and their bootstrap support are not meaningful
- Working with rDNA metabarcode markers (16S, ITS) whose accuracy shows distinctive stem-loop variation rather than the length-insensitive behavior of CO1
- You need species-rank assignments — no bootstrap cutoff yields 99% correct species assignments (NA in Table 2); fall back to genus or more inclusive ranks

## Inputs

- CO1 Eukaryote v1 training set (taxonomy and FASTA files from the CO1Classifier GitHub release)
- Query CO1 sequences truncated or subsampled to target fragment lengths (500 bp+, 400 bp, 200 bp, 100 bp, 50 bp)
- Known taxonomy for each sequence (for leave-one-out correctness scoring)

## Outputs

- Bootstrap support proportion (0–1) per taxonomic assignment per sequence
- Matrix of minimum bootstrap support cutoffs per taxonomic rank and fragment length for 99% correct assignments
- accuracy_by_fragment_length.csv with proportion of correct assignments per rank and fragment length

## How to apply

Classify each sequence with the RDP classifier v2.12 using the CO1 Eukaryote v1 training set (singleton taxa excluded, since assignments are only meaningful for taxa represented in the database). For each query, randomly subsample the set of 8 bp words and repeat the classification 100 times, recording the proportion of times the original assignment is recovered as the bootstrap support. Run leave-one-out testing, in which each sequence is removed from the training set, classified, and scored as correct when the assignment matches its known taxonomy; tally correct assignments per taxonomic rank and per fragment length (500 bp+, 400 bp, 200 bp, 100 bp, 50 bp). Finally, calculate a matrix of minimum bootstrap support cutoffs that yields 99% correct assignments — e.g. the 70% cutoff selected for genus-rank assignments with full-length (500 bp+) CO1 sequences — and write the per-rank/per-length accuracy summary to accuracy_by_fragment_length.csv.

## Related tools

- **RDP classifier** (Naïve Bayesian classifier run with the CO1 Eukaryote v1 training set for leave-one-out testing and 100× bootstrap re-classification from subsampled 8 bp words) — https://github.com/terrimporter/CO1Classifier
- **CO1 Eukaryote v1 training set** (Reference taxonomy and FASTA files used to train the RDP classifier and derive bootstrap cutoffs) — https://github.com/terrimporter/CO1Classifier/releases

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Bootstrap support for each assignment equals the proportion of the 100 randomized 8 bp word subsampling runs recovering the original assignment
- Tabulated accuracy pattern shows performance similar across CO1 fragment lengths, with highest accuracy at more inclusive ranks, especially for fragments 200 bp or longer (consistent with Table 2)
- Minimum bootstrap cutoff for genus-rank assignments with 500 bp+ sequences is 70%, matching the 99% correct-assignment matrix
- Singleton taxa are excluded from leave-one-out results before tallying accuracy
- Output CSV (accuracy_by_fragment_length.csv) contains one row per rank × fragment length with proportions in [0,1]

## Limitations

- No bootstrap cutoff achieves 99% correct species-rank assignments (NA entries in the Table 2 matrix)
- Classifier accuracy and cutoffs apply only when query sequences (or close relatives) are present in the reference set; missing congeneric sequences cause false negatives
- Method has a high false negative rate in practice, so the README offers more lenient cutoffs for users accepting ~90% correct assignments

## Evidence

- [methods] used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [results] calculated a matrix of minimum bootstrap support value cutoffs to obtain 99% correct assignments: [evidence span withheld — anchor status: not_found]
- [results] 70% bootstrap support cutoff selected for full length CO1 sequences: "The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2."
- [results] Classifier accuracy is highest at more inclusive taxonomic ranks: [evidence span withheld — anchor status: not_found]
- [readme] README minimum bootstrap support cutoffs for 99% correct assignments: "using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
