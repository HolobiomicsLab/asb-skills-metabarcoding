---
name: taxonomic-sequence-classification
description: Use when assigning genus-rank taxonomy to primer-anchored CO1 amplicon
  sequences (~100-650 bp) and you need high-confidence assignments with a controlled
  false positive rate rather than accepting the top BLAST hit.
when_to_use_negative:
- 'You need species-rank assignments for short fragments: Table 2 gives NA (no cutoff
  achieves 99% correct assignments) for species rank, so use genus or more inclusive
  ranks for 200 bp reads'
- 'Query sequences are expected to be absent from the reference database (e.g. quarantine
  screening for unstudied local biota): the classifier cannot classify taxa not in
  the training set and false negatives occur when congeneric sequences are missing'
- Your input is already an OTU/ASV feature table with taxonomy — no per-sequence classification
  step is needed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: RDP classifier
  role: Naive Bayesian classifier used to assign CO1 query sequences to genus rank
    with genus-rank bootstrap support cutoffs (0.60 for 200 bp, 0.70 for 500 bp+)
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: blastn/megablast against a custom CO1 nucleotide database to implement the
    top BLAST hit comparison method (pident >= 95%, qcovs >= 85%)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:8cfb34ecc567628d23b3f62f8cceacad4f664d6b351addbf78b834bd0b017dd1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - RDP classifier
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# taxonomic-sequence-classification

## Summary

Assign taxonomy to CO1 metabarcode sequences using the RDP naive Bayesian classifier with pre-trained CO1 Eukaryote reference sets, and benchmark it against the top BLAST hit method; the RDP approach reduces the false positive rate from nearly 100% (top BLAST hit) to 34% while running ~19x faster.

## When to use

Use when assigning genus-rank taxonomy to primer-anchored CO1 amplicon sequences (~100-650 bp) and you need high-confidence assignments with a controlled false positive rate rather than accepting the top BLAST hit.

## When NOT to use

- You need species-rank assignments for short fragments: Table 2 gives NA (no cutoff achieves 99% correct assignments) for species rank, so use genus or more inclusive ranks for 200 bp reads
- Query sequences are expected to be absent from the reference database (e.g. quarantine screening for unstudied local biota): the classifier cannot classify taxa not in the training set and false negatives occur when congeneric sequences are missing
- Your input is already an OTU/ASV feature table with taxonomy — no per-sequence classification step is needed

## Inputs

- Query CO1 metabarcode sequences in FASTA format (e.g. primer-anchored 200 bp fragments)
- Trained RDP classifier files: CO1 Eukaryote v1 trained set and rRNAClassifier.properties from the CO1Classifier release
- Custom CO1 reference nucleotide database for BLAST comparison

## Outputs

- rdp.output file with genus-rank taxonomic assignments and bootstrap support values
- BLAST hit table (pident, qcovs) per query sequence
- Set of high-confidence assignments per method (RDP bootstrap >= 0.60 at genus rank; BLAST pident >= 95% and qcovs >= 85%)
- False positive rate comparison table (e.g. fpr_comparison.csv): top BLAST hit ~100% vs RDP 34%

## How to apply

Download a trained CO1 classifier release (e.g. CO1 Eukaryote v1 trained set plus rRNAClassifier.properties) from github.com/terrimporter/CO1Classifier and run the RDP classifier v2.12: 'java -Xmx8g -jar classifier.jar classify -t rRNAClassifier.properties -o rdp.output query.fasta', recording genus-rank bootstrap support. Apply the 'good' assignment criterion from Table 2: a bootstrap proportion of 0.60 or greater at the genus rank for 200 bp fragments (70% for full-length 500 bp+ sequences). For comparison, run blastn (megablast, evalue 1e-20, outfmt with pident and qcovs) against a local CO1 nucleotide database and accept a top hit only if percent identity >= 95% and alignment spans >= 85% of the original query length. Rationale: the top BLAST hit method implicitly assumes the query is present in the database, so database annotation errors propagate; the RDP classifier's bootstrap cutoffs are calibrated (via leave-one-out / cross-validation testing) so that assignments meeting the cutoff are at least ~99% correct.

## Related tools

- **RDP classifier** (Naive Bayesian classifier used to assign CO1 query sequences to genus rank with genus-rank bootstrap support cutoffs (0.60 for 200 bp, 0.70 for 500 bp+)) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (blastn/megablast against a custom CO1 nucleotide database to implement the top BLAST hit comparison method (pident >= 95%, qcovs >= 85%))

## Examples

```
java -Xmx8g -jar classifier.jar classify -t rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- Reproduced comparison shows the RDP classifier reduces the false positive rate from nearly 100% (top BLAST hit) to about 34%
- RDP output contains a genus-rank bootstrap value for each query; retained assignments all have bootstrap >= 0.60 (200 bp) or >= 0.70 (500 bp+)
- BLAST 'good' hits satisfy both pident >= 95% and alignment coverage >= 85% of the original query length
- Assignment rate near 99% correct is expected when queries are present in the reference set and the Table 2 cutoffs are applied
- Runtime check: RDP assignments complete roughly 19x faster than the top BLAST hit method

## Limitations

- False negatives occur when congeneric sequences are missing from the database, a particular concern in quarantine situations
- No cutoff achieves 99% correct species-rank assignments for short fragments (NA in Table 2); accuracy is highest at more inclusive ranks, especially for fragments 200 bp or longer
- CO1 reference database coverage is incomplete — only 22% of extant Insecta genera have full-length species-rank CO1 barcodes in GenBank
- README cutoffs for newer training sets differ from the article's Table 2 values (e.g. v5.1.0 recommends genus 0.3 for 200 bp), so cutoffs must match the classifier version used
- Top BLAST hit false positives stem from misannotated database records; the method assumes the query sequence is actually represented in the database

## Evidence

- [abstract] RDP classifier reduces the false positive rate from nearly 100% to 34%: "reduce the false positive rate from nearly 100% to 34%"
- [results] 60% bootstrap cutoff for primer-anchored 200 bp sequences: [evidence span withheld — anchor status: not_found]
- [methods] Good assignments defined per Table 2 for 200 bp fragments at genus rank: "‘Good’ assignments for the RDP classifier were defined according to Table 2 for 200 bp fragments at the genus rank"
- [discussion] Classifier v2.12 with CO1 Eukaryote v1 training set: "the RDP classifier 2.12 with the CO1 Eukaryote v1 training set"
- [readme] README invocation of the RDP classifier with trained properties file: "rdp_classifier -Xmx8g classify -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta"
