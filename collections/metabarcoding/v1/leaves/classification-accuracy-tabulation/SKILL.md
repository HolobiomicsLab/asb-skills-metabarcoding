---
name: classification-accuracy-tabulation
description: Use when evaluating a trained taxonomic classifier (e.g., RDP classifier
  with a CO1 training set) to determine how assignment accuracy changes with query
  fragment length and rank, or to pick bootstrap support cutoffs.
when_to_use_negative:
- The query sequences are novel metabarcoding reads rather than training-set members
  with known taxonomy — leave-one-out testing is a benchmarking procedure, not an
  assignment pipeline.
- You need species-rank accuracy for fragments shorter than 500 bp where no cutoff
  achieves 99% correct assignments (NA in the cutoff matrix).
- The classifier training set contains only singleton taxa for the ranks of interest,
  since assignments are not meaningful for taxa unrepresented (or represented once)
  in the database.
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: RDP classifier v2.12
  role: Naïve Bayesian classifier used for leave-one-out testing, cross-validation
    testing, and classifier training; classifies each held-out sequence using 8 bp
    words with 100-replicate bootstrap subsampling.
  repo: https://github.com/terrimporter/CO1Classifier
- name: CO1 Eukaryote v1 training set
  role: Reference taxonomy and FASTA training files providing the sequences and known
    labels benchmarked across fragment lengths.
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:42d662b71001879c9b7eb15a45a2a283acadfdd2d1b55e3763aa6fc7fd982785
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
yes_rate: 0.9230769230769231
n_claims: 13
---

# classification-accuracy-tabulation

## Summary

A leave-one-out benchmarking protocol that quantifies how RDP classifier taxonomic assignment accuracy for CO1 metabarcodes varies with sequence fragment length (500bp+, 400, 200, 100, 50 bp) and taxonomic rank, producing a table of correct-assignment proportions and minimum bootstrap support cutoffs.

## When to use

Use when evaluating a trained taxonomic classifier (e.g., RDP classifier with a CO1 training set) to determine how assignment accuracy changes with query fragment length and rank, or to pick bootstrap support cutoffs.

## When NOT to use

- The query sequences are novel metabarcoding reads rather than training-set members with known taxonomy — leave-one-out testing is a benchmarking procedure, not an assignment pipeline.
- You need species-rank accuracy for fragments shorter than 500 bp where no cutoff achieves 99% correct assignments (NA in the cutoff matrix).
- The classifier training set contains only singleton taxa for the ranks of interest, since assignments are not meaningful for taxa unrepresented (or represented once) in the database.

## Inputs

- CO1 Eukaryote v1 training set: taxonomy file and FASTA file from the CO1Classifier GitHub release (github.com/terrimporter/CO1Classifier)
- Full-length (500 bp+) CO1 reference sequences with known taxonomy (training set members, singletons excluded)
- Target fragment lengths: 500bp+, 400 bp, 200 bp, 100 bp, 50 bp

## Outputs

- accuracy_by_fragment_length.csv: proportion of correct assignments per taxonomic rank per fragment length
- Matrix of minimum bootstrap support cutoffs per rank per fragment length for 99% correct assignments (Table 2 analogue)
- Per-assignment bootstrap support values (proportion of 100 subsampled-word classification replicates recovering the original assignment)

## How to apply

Load the CO1 Eukaryote v1 training set (taxonomy and FASTA files from the CO1Classifier GitHub release) and exclude singleton taxa, since classifier assignments are only meaningful for taxa represented in the database. For each fragment length condition (500bp+, 400, 200, 100, 50 bp), truncate or subsample sequences to the target length and run leave-one-out testing with the RDP classifier v2.12: each sequence is removed from the training set, classified using a full set of 8 bp words subsampled from the query, then returned. Assess bootstrap support by randomly subsampling 8 bp words from the query and repeating the classification 100 times, recording the proportion of times the original assignment is recovered. Score each assignment as correct when it matches the known taxonomy, tally correct assignments per taxonomic rank per fragment length, and write the summary to accuracy_by_fragment_length.csv. Additionally tabulate a matrix of minimum bootstrap support cutoffs that yield ~99% correct assignments (e.g., genus rank 40 for 500bp+ and 90 for 50 bp in the v3 cutoffs) to mirror the article's Table 2.

## Related tools

- **RDP classifier v2.12** (Naïve Bayesian classifier used for leave-one-out testing, cross-validation testing, and classifier training; classifies each held-out sequence using 8 bp words with 100-replicate bootstrap subsampling.) — https://github.com/terrimporter/CO1Classifier
- **CO1 Eukaryote v1 training set** (Reference taxonomy and FASTA training files providing the sequences and known labels benchmarked across fragment lengths.) — https://github.com/terrimporter/CO1Classifier

## Examples

```
java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- The tabulated accuracy pattern shows classification performance similar across the CO1 barcoding region length (500bp+ down to 50 bp), consistent with the article's reported leave-one-out results.
- Accuracy is highest at more inclusive taxonomic ranks, especially for fragments 200 bp or longer.
- Singleton taxa are excluded from leave-one-out results before tallying correct assignments.
- Bootstrap support values fall in [0, 1] and equal the proportion of 100 subsampled 8 bp-word replicates recovering the original assignment.
- Minimum bootstrap cutoffs per rank/length reproduce the expected monotonic pattern (e.g., 0 at superkingdom/kingdom/phylum for all lengths; genus 40 for 500bp+ rising to 90 for 50 bp in the v3 matrix).

## Limitations

- The classifier is not meant to classify taxa not represented in the database, so leave-one-out results from singletons must be excluded.
- No cutoff is available (NA) that yields 99% correct assignments at the species rank; species-level accuracy should be assessed at the ~95% level instead.
- CO1 reference database coverage is incomplete (e.g., only 22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank), so accuracy estimates reflect database gaps.
- The method has been shown to have a high false negative rate in practice, so lenient cutoffs yielding ~90% correct assignments may be preferred.

## Evidence

- [abstract] Classification performance is similar across the length of the CO1 barcoding region: "we show that classification performance is similar across the length of the CO1 barcoding region"
- [methods] RDP classifier v2.12 used for leave-one-out testing: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [results] Singletons excluded from leave-one-out results: "Since the classifier is not meant to classify taxa not represented in the database, leave-one-out testing results from singletons were excluded"
- [results] Matrix of minimum bootstrap support cutoffs for 99% correct assignments: [evidence span withheld — anchor status: not_found]
- [results] Accuracy highest at more inclusive ranks for longer fragments: [evidence span withheld — anchor status: not_found]
- [readme] README v3 cutoff table with 50 bp column (genus 90): "Rank | 500bp+ | 400 bp | 200 bp | 100 bp | 50 bp  
--- |:---:|:---:|:---:|:---:|:---:  
Superkingdom | 0 | 0 | 0 | 0 | 0"
