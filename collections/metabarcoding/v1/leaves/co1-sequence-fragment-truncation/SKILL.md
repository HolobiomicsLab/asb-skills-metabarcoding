---
name: co1-sequence-fragment-truncation
description: Use when evaluating how CO1 metabarcode classification accuracy of the
  RDP classifier changes with fragment length (e.g., 500bp+ down to 50 bp) or taxonomic
  rank, to set bootstrap support cutoffs for short-read metabarcoding.
when_to_use_negative:
- The query sequences are full-length 500bp+ barcodes only and you need a production
  assignment rather than a length-benchmark — use a single classification run with
  the 70% genus-rank bootstrap cutoff instead.
- The benchmark targets rDNA metabarcode markers (e.g., 16S, ITS) rather than CO1
  — the article notes rDNA markers show distinctive stem-loop variation in accuracy
  that this CO1-specific benchmark does not model.
- You need the newer 5-fold cross-validation cutoffs of the v5.1.0 release rather
  than reproducing the original v1 leave-one-out benchmark.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: RDP classifier v2.12
  role: Runs leave-one-out testing, cross-validation testing, and classifier training
    on truncated/subsampled CO1 fragments using 8 bp words with 100-replicate bootstrap
    support estimation
  repo: https://github.com/terrimporter/CO1Classifier
- name: CO1 Eukaryote v1 training set
  role: Source taxonomy and FASTA reference files that are truncated/subsampled per
    fragment length condition
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:ed59721db20f9048dde876548b4d96a3ce77fa9eaf15098a685f845628387a53
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - CO1 Eukaryote v1 training set
  - RDP classifier v2.12
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# co1-sequence-fragment-truncation

## Summary

Truncates or subsamples CO1 reference sequences to metabarcode-scale fragment lengths (500bp+, 400, 200, 100, 50 bp) and runs leave-one-out testing with the RDP classifier v2.12 to benchmark how classification accuracy varies with fragment length and taxonomic rank.

## When to use

Use when evaluating how CO1 metabarcode classification accuracy of the RDP classifier changes with fragment length (e.g., 500bp+ down to 50 bp) or taxonomic rank, to set bootstrap support cutoffs for short-read metabarcoding.

## When NOT to use

- The query sequences are full-length 500bp+ barcodes only and you need a production assignment rather than a length-benchmark — use a single classification run with the 70% genus-rank bootstrap cutoff instead.
- The benchmark targets rDNA metabarcode markers (e.g., 16S, ITS) rather than CO1 — the article notes rDNA markers show distinctive stem-loop variation in accuracy that this CO1-specific benchmark does not model.
- You need the newer 5-fold cross-validation cutoffs of the v5.1.0 release rather than reproducing the original v1 leave-one-out benchmark.

## Inputs

- CO1 Eukaryote v1 training set taxonomy file (from the CO1Classifier GitHub release)
- CO1 Eukaryote v1 training set FASTA file (from the CO1Classifier GitHub release)
- Target fragment length conditions (500bp+, 400 bp, 200 bp, 100 bp, 50 bp)

## Outputs

- accuracy_by_fragment_length.csv — proportion of correct assignments per taxonomic rank per fragment length
- Matrix of minimum bootstrap support cutoffs yielding 99% correct assignments (mirroring Table 2)

## How to apply

Load the CO1 Eukaryote v1 training set (taxonomy and FASTA files from the CO1Classifier GitHub release) and exclude singleton taxa, since assignments are only meaningful for taxa represented in the database. For each fragment length condition (500bp+, 400 bp, 200 bp, 100 bp, 50 bp), truncate or subsample the sequences to the target length and run leave-one-out testing with the RDP classifier v2.12: remove each sequence from the training set, classify it using a full set of 8 bp words subsampled from the query, then return it. Assess bootstrap support by randomly subsampling 8 bp words from the query and repeating the classification 100 times, recording how often the original assignment is recovered. Score each assignment as correct when it matches the sequence's known taxonomy, tally correct assignments per taxonomic rank per fragment length, and write the summary to accuracy_by_fragment_length.csv.

## Related tools

- **RDP classifier v2.12** (Runs leave-one-out testing, cross-validation testing, and classifier training on truncated/subsampled CO1 fragments using 8 bp words with 100-replicate bootstrap support estimation) — https://github.com/terrimporter/CO1Classifier
- **CO1 Eukaryote v1 training set** (Source taxonomy and FASTA reference files that are truncated/subsampled per fragment length condition) — https://github.com/terrimporter/CO1Classifier

## Examples

```
rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query_200bp.fasta
```

## Evaluation signals

- Tabulated accuracy shows classification performance similar across the length of the CO1 barcoding region (500bp+ through 50 bp), consistent with Table 2.
- Accuracy is highest at more inclusive taxonomic ranks, especially for fragments 200 bp or longer.
- Singleton taxa are excluded from leave-one-out results before tallying correct assignments.
- Bootstrap support is estimated by 100 replicate classifications with random subsampling of 8 bp words from the query.
- Summary is written to accuracy_by_fragment_length.csv with proportions of correct assignments per rank per fragment length.

## Limitations

- Species-rank assignments had no bootstrap cutoff available that yielded 99% correct assignments (NA in the cutoff matrix); 95% correct assignments at species rank required cutoffs such as 60 (500bp+) to NA (50 bp) in the v3 tables.
- The classifier is not meant to classify taxa not represented in the database, so leave-one-out results from singletons were excluded.
- Findings are specific to CO1; rDNA metabarcode markers show distinctive stem-loop variation in accuracy across fragment lengths.
- Bootstrap cutoffs are tied to the training set version — the README reports different cutoffs for v3 (leave-one-out style) versus v5.1.0 (5-fold cross validation), so cutoffs must not be mixed across versions.

## Evidence

- [results] Leave-one-out testing with fragment truncation: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [results] Singleton exclusion rationale: "Since the classifier is not meant to classify taxa not represented in the database, leave-one-out testing results from singletons were excluded"
- [abstract] Fragment-length invariance finding: "we show that classification performance is similar across the length of the CO1 barcoding region"
- [results] Rank-dependent accuracy pattern: [evidence span withheld — anchor status: not_found]
- [results] Bootstrap cutoff matrix: [evidence span withheld — anchor status: not_found]
- [readme] Version-specific cutoffs in README: "Assuming that your query sequences are present in the reference set, using these cutoffs should result in ~99% correct assignments"
