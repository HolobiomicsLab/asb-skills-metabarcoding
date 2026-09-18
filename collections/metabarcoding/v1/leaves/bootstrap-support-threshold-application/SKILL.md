---
name: bootstrap-support-threshold-application
description: Use when assigning taxonomy to CO1 metabarcode sequences with the RDP
  classifier and a bootstrap support threshold must be selected to match fragment
  length and target rank (e.g., genus vs species).
when_to_use_negative:
- Query sequences are not represented in the reference/training set — high bootstrap
  support cannot be expected and false negatives will dominate
- Assignments at the species rank are required with the genus-trained CO1 Eukaryote
  v1 classifier — no genus-rank cutoff guarantees species-level accuracy (species
  is NA for 99% correct assignments)
- The taxonomic assignment method is top BLAST hit rather than the RDP classifier
  — bootstrap support cutoffs do not apply
edam_operation: http://edamontology.org/operation_0372
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: RDP classifier
  role: Naive Bayesian classifier whose genus-rank bootstrap support values are thresholded
    with rank- and length-specific cutoffs
  repo: https://github.com/terrimporter/CO1Classifier
- name: RDP classifier 2.12
  role: Version used with the CO1 Eukaryote v1 training set to generate bootstrap
    support for leave-one-out and cross-validation testing that defined the cutoffs
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:540a1da703578bc386edd7a15134ef9e7d3bf34f92fbdc4ee0a8a88d6d5a2fcc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - RDP classifier
  - RDP classifier 2.12
  edam_operation: http://edamontology.org/operation_0372
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# bootstrap-support-threshold-application

## Summary

Applies rank- and fragment-length-specific RDP classifier bootstrap support cutoffs to control false positive rates in CO1 metabarcode taxonomic assignment. The article's Table 2 and the CO1Classifier README define minimum bootstrap proportions (e.g., 0.60 at genus rank for primer-anchored 200 bp fragments) that yield ~99% correct assignments.

## When to use

Use when assigning taxonomy to CO1 metabarcode sequences with the RDP classifier and a bootstrap support threshold must be selected to match fragment length and target rank (e.g., genus vs species).

## When NOT to use

- Query sequences are not represented in the reference/training set — high bootstrap support cannot be expected and false negatives will dominate
- Assignments at the species rank are required with the genus-trained CO1 Eukaryote v1 classifier — no genus-rank cutoff guarantees species-level accuracy (species is NA for 99% correct assignments)
- The taxonomic assignment method is top BLAST hit rather than the RDP classifier — bootstrap support cutoffs do not apply

## Inputs

- CO1 query sequences in FASTA format (e.g., primer-anchored 200 bp metabarcodes or full-length 500 bp+ barcodes)
- RDP classifier output with bootstrap support values per rank (rdp.output)
- CO1 training set release files (e.g., CO1 Eukaryote v1 trained set with rRNAClassifier.properties, or RDP_COIv5.1.0.zip)
- Bootstrap cutoff matrix (Table 2 of the article, or the rank-by-length cutoff tables in the README)

## Outputs

- Filtered list of high-confidence genus-rank (or other rank) taxonomic assignments passing the bootstrap cutoff
- Assignment outcome tallies (correct, incorrect, unassigned) usable to compute false positive/false negative rates, e.g., in fpr_comparison.csv

## How to apply

Run the RDP classifier (v2.12 or v2.13) with a CO1-trained rRNAClassifier.properties file from the terrimporter/CO1Classifier release, recording the bootstrap proportion for each rank. Select the cutoff from the matrix of minimum bootstrap support values calculated to obtain 99% correct assignments during leave-one-out testing: for the original CO1 Eukaryote v1 set this is 70% at the genus rank for full-length 500 bp+ sequences and 60% at the genus rank for primer-anchored 200 bp fragments; for later v5.1.0 releases the 99%-correct genus cutoff is 0.3 across 100 bp to 500 bp+ lengths. Retain only assignments whose genus-rank bootstrap proportion meets or exceeds the chosen cutoff, and summarize assignments to genus or more inclusive ranks to mitigate database annotation errors. Judge application correct when the filtered assignments reproduce the expected accuracy/false-positive behavior (RDP classifier false positive rate of about 34% versus nearly 100% for the top BLAST hit method).

## Related tools

- **RDP classifier** (Naive Bayesian classifier whose genus-rank bootstrap support values are thresholded with rank- and length-specific cutoffs) — https://github.com/terrimporter/CO1Classifier
- **RDP classifier 2.12** (Version used with the CO1 Eukaryote v1 training set to generate bootstrap support for leave-one-out and cross-validation testing that defined the cutoffs) — https://github.com/terrimporter/CO1Classifier

## Examples

```
java -Xmx8g -jar classifier.jar classify -t rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Filtered genus-rank assignments all have bootstrap proportion >= the selected cutoff (0.60 for 200 bp fragments per Table 2; 0.3 for v5.1.0 across lengths)
- Reproduced false positive rate for the RDP classifier on primer-anchored 200 bp sequences is about 34%, versus nearly 100% for the top BLAST hit method
- Assignment accuracy at the applied cutoff matches the README expectation of at least 99% correct assignments for sequences present in the reference set
- Cutoff selection matches the rank-by-fragment-length matrix (e.g., 70% genus cutoff for 500 bp+ full-length CO1 in the original Table 2)
- Low-support sequences are reported as unassigned rather than forced to a genus, keeping the false negative rate documented and interpretable

## Limitations

- Cutoffs guarantee ~99% correct assignments only when query sequences are actually present in the reference set; database gaps cause false negatives when congeneric sequences are missing
- No genus-rank cutoff is available (NA) that will result in 99% correct species-rank assignments with the genus-trained classifier
- The method has a high false negative rate in practice, so lenient cutoff tables (~90% correct assignments) are provided as an alternative
- Cutoff values differ between training set versions (v1/v3 vs v5.1.0), so the table must match the training set actually used

## Evidence

- [results] The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2.: "The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2."
- [results] 60% bootstrap cutoff for 200 bp: [evidence span withheld — anchor status: not_found]
- [readme] README cutoff guarantee: "Assuming that your query sequences are present in the reference set, using these minimum bootstrap support cutoffs should result in at least 99% correct assignments"
- [abstract] false positive rate reduction: "reduce the false positive rate from nearly 100% to 34%"
