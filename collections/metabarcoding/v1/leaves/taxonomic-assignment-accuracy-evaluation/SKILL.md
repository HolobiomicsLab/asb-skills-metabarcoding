---
name: taxonomic-assignment-accuracy-evaluation
description: Use when you have a CO1 (or similar marker-gene) taxonomy+FASTA reference
  training set and need to quantify classifier accuracy or calibrate minimum bootstrap
  support cutoffs per taxonomic rank and sequence length.
when_to_use_negative:
- Query sequences are not represented in the reference set — the 99% correct-assignment
  guarantee explicitly assumes 'your query sequences are present in the reference
  set'
- The taxon of interest is a singleton or absent from the training database — leave-one-out
  results for singletons are excluded and the classifier cannot classify unrepresented
  taxa
- Applying paper/Table 2 cutoffs to a different training-set version — cutoffs are
  version-specific (v1 vs v3 vs v5.1.0 differ substantially) and must be matched to
  the release used
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: RDP classifier
  role: Naïve Bayesian classifier run in leave-one-out mode with 100-fold bootstrap
    subsampling of 8 bp words to estimate assignment support and calibrate rank-specific
    cutoffs
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Top BLAST hit method used as the comparison baseline (RDP classifier was ~19x
    faster and reduced false positives from nearly 100% to 34%)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:110519409c8eb777e6b0c9dd793c9a5c42b034cae6f6beba97f94320228ad81d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# taxonomic-assignment-accuracy-evaluation

## Summary

Evaluates the taxonomic assignment accuracy of an RDP-classifier-based CO1 metabarcode classifier by leave-one-out testing over a reference training set (e.g. CO1 Eukaryote v1), estimating bootstrap support by 100-fold random subsampling of 8 bp words, and deriving rank- and fragment-length-specific minimum bootstrap cutoffs that yield 99% correct assignments.

## When to use

Use when you have a CO1 (or similar marker-gene) taxonomy+FASTA reference training set and need to quantify classifier accuracy or calibrate minimum bootstrap support cutoffs per taxonomic rank and sequence length.

## When NOT to use

- Query sequences are not represented in the reference set — the 99% correct-assignment guarantee explicitly assumes 'your query sequences are present in the reference set'
- The taxon of interest is a singleton or absent from the training database — leave-one-out results for singletons are excluded and the classifier cannot classify unrepresented taxa
- Applying paper/Table 2 cutoffs to a different training-set version — cutoffs are version-specific (v1 vs v3 vs v5.1.0 differ substantially) and must be matched to the release used

## Inputs

- Reference training set taxonomy file (e.g. CO1 Eukaryote v1 taxonomy from github.com/terrimporter/CO1Classifier)
- Reference training set FASTA file of CO1 sequences (500 bp+, ambiguity-free, screened for contaminants)
- Query FASTA files of CO1 sequences or fragments (full-length 500 bp+, 400 bp, 200 bp, 100 bp, 50 bp)
- RDP classifier v2.12 (trained properties file, e.g. rRNAClassifier.properties)

## Outputs

- Matrix of minimum bootstrap support cutoffs per taxonomic rank and fragment length achieving 99% correct assignments (table2_bootstrap_cutoffs.csv; paper Table 2)
- Per-query assignment outcomes and bootstrap support values from leave-one-out testing
- Proportion of correct assignments per rank and fragment length

## How to apply

Load the reference training set (taxonomy file + FASTA, e.g. CO1 Eukaryote v1 from github.com/terrimporter/CO1Classifier) and run leave-one-out testing with the RDP classifier v2.12, classifying each removed sequence from full-length (500 bp+) sequences and 400, 200, 100, and 50 bp fragments, using a full set of 8 bp 'words' subsampled from the query. For every query, estimate bootstrap support by randomly subsampling the 8 bp words, re-assigning, and repeating 100 times; score an assignment as correct when it matches the known taxonomy. Exclude singleton taxa from leave-one-out results since the classifier cannot classify taxa absent from the database. Aggregate outcomes across bootstrap support thresholds for each taxonomic rank x fragment length combination and select the minimum cutoff per cell that yields at least 99% correct assignments (paper Table 2: genus rank 70% for 500 bp+, 60% for 400/200/100 bp, N/A for 50 bp). Note that cutoffs are training-set-version-specific: the README's v3 and v5.1.0 tables report different values (e.g. genus 0.3 across lengths for v5.1.0, cross-validated), so always pair cutoffs with the matching release.

## Related tools

- **RDP classifier** (Naïve Bayesian classifier run in leave-one-out mode with 100-fold bootstrap subsampling of 8 bp words to estimate assignment support and calibrate rank-specific cutoffs) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Top BLAST hit method used as the comparison baseline (RDP classifier was ~19x faster and reduced false positives from nearly 100% to 34%))

## Examples

```
java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Each reported minimum cutoff must achieve at least 99% correct assignments in the leave-one-out test, matching the paper's Table 2 criterion
- Cutoffs should increase monotonically with shorter fragments and more specific taxonomic ranks (e.g. genus: 70% at 500 bp+ dropping then N/A at 50 bp in Table 2)
- Singleton taxa are excluded from leave-one-out aggregates and no cutoff is reported for cells marked NA (e.g. species rank, 50 bp genus)
- Bootstrap support per query is estimated from 100 random subsamples of the full 8 bp word set, so reproducibility requires the same word-subsampling scheme
- Reported cutoffs must match the training-set release: genus 70% (500 bp+) for CO1 Eukaryote v1 vs 0.3 across lengths in the README v5.1.0 cross-validated table

## Limitations

- No cutoff achieves 99% correct assignments at species rank (NA) and at the shortest fragments (e.g. genus at 50 bp in Table 2); species-level work requires accepting ~95% accuracy (e.g. species cutoff 60% at 500 bp+ in the v3 README table)
- The method has a high false negative rate in practice, so the README also offers more lenient cutoffs targeting ~90% correct assignments
- Cutoffs assume the query sequence is present in the reference set; false negatives occur when congeneric sequences are missing from the database
- Newer training set versions (v5/v5.1.0) assess accuracy by 5-fold cross validation rather than leave-one-out, so cutoff values are not directly comparable to the paper's Table 2

## Evidence

- [results] we have calculated a matrix of minimum bootstrap support value cutoffs to obtain 99% correct assignments during leave one out testing (Table 2).: [evidence span withheld — anchor status: not_found]
- [results] The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2.: "The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2."
- [methods] used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training: "used with the RDP classifier v 2.12 for leave-one-out testing, cross-validation testing, and classifier training"
- [readme] Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
- [results] leave-one-out testing results from singletons were excluded from this figure and no bootstrap support cutoff was: "leave-one-out testing results from singletons were excluded from this figure and no bootstrap support cutoff was"
