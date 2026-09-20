---
name: co1-reference-database-deployment
description: Use when CO1 (cox1/COI) metabarcode amplicons need genus- or species-rank
  taxonomic assignment using a curated eukaryote reference set and the RDP classifier
  instead of top BLAST hit assignment.
when_to_use_negative:
- The query sequences are not CO1 (the training sets are built for eukaryote CO1 mtDNA;
  use SILVA, Greengenes, or UNITE for 16S/ITS data)
- A web portal is required — no portal exists yet for this RDP-based CO1 method; the
  CLI is the only route
- Species-rank assignment is needed with the CO1 Eukaryote v1 training set, for which
  no species-rank cutoff is available (NA)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0080
tools:
- name: RDP classifier
  role: Naïve Bayesian classifier that assigns CO1 query sequences to taxonomy ranks
    using the pre-trained rRNAClassifier.properties training set
  repo: https://github.com/terrimporter/CO1Classifier
- name: BLAST
  role: Alternative top-hit assignment engine; BLAST-formatted reference sequences
    are also shipped in releases since v5.0.0 for comparison workflows
  repo: https://github.com/terrimporter/CO1Classifier
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-018-22505-4
    title: Automated high throughput animal CO1 metabarcode classification
  source_content_hash: sha256:b0a240dbb16e18a6c429d0fc5a3a9e53da16ffeac0bedf609ea97cad4710a45a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/terrimporter/CO1Classifier
  tools:
  - BLAST
  - RDP classifier
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# co1-reference-database-deployment

## Summary

Deploy the pre-trained CO1 Eukaryote reference training sets from the terrimporter/CO1Classifier repository with the RDP classifier (v2.12/2.13) to assign taxonomy to CO1 metabarcode sequences, and apply the correct length- and rank-specific bootstrap support cutoffs.

## When to use

Use when CO1 (cox1/COI) metabarcode amplicons need genus- or species-rank taxonomic assignment using a curated eukaryote reference set and the RDP classifier instead of top BLAST hit assignment.

## When NOT to use

- The query sequences are not CO1 (the training sets are built for eukaryote CO1 mtDNA; use SILVA, Greengenes, or UNITE for 16S/ITS data)
- A web portal is required — no portal exists yet for this RDP-based CO1 method; the CLI is the only route
- Species-rank assignment is needed with the CO1 Eukaryote v1 training set, for which no species-rank cutoff is available (NA)

## Inputs

- CO1 metabarcode query sequences in FASTA format (e.g. primer-anchored 200 bp fragments or 500 bp+ amplicons)
- Trained RDP classifier files (rRNAClassifier.properties plus accompanying trained data) from a CO1Classifier release
- Reference files for BLAST/SINTAX formatting (optional, available as of version 5.0.0)

## Outputs

- RDP classifier output file (rdp.output) with taxonomic assignments and bootstrap support values per query per rank
- High-confidence genus- or species-rank assignments after applying the length-specific bootstrap cutoff

## How to apply

Download the latest trained release (e.g. RDP_COIv5.1.0.zip) from github.com/terrimporter/CO1Classifier/releases, which contains the ready-to-use rRNAClassifier.properties file; no further training is needed from the user. Install the RDP classifier (conda install -c bioconda rdp_classifier or the SourceForge zip) and run classification with java -Xmx8g -jar classifier.jar classify -t rRNAClassifier.properties -o rdp.output query.fasta. Choose the minimum bootstrap support cutoff from the release's cutoff table based on query length: the paper's CO1 Eukaryote v1 set uses a genus-rank cutoff of 60% for primer-anchored 200 bp fragments (Table 2) and 70% for 500 bp+ sequences, while newer v5.1.0 cross-validated cutoffs list 0.3 at the genus rank. Rationale: assignments below the cutoff are treated as unassigned because the classifier has a high false negative rate in practice, but honoring the cutoff keeps correct assignments at or above ~99% for queries represented in the reference set. Version choice matters: v1 supports genus-rank only (species rank is NA), while later releases (v3+, v5.1.0) add species-rank support and reference sequences formatted for BLAST and SINTAX.

## Related tools

- **RDP classifier** (Naïve Bayesian classifier that assigns CO1 query sequences to taxonomy ranks using the pre-trained rRNAClassifier.properties training set) — https://github.com/terrimporter/CO1Classifier
- **BLAST** (Alternative top-hit assignment engine; BLAST-formatted reference sequences are also shipped in releases since v5.0.0 for comparison workflows) — https://github.com/terrimporter/CO1Classifier

## Examples

```
java -Xmx8g -jar classifier.jar classify -t rRNAClassifier.properties -o rdp.output query.fasta
```

## Evaluation signals

- The rRNAClassifier.properties file from the release decompresses cleanly and the classifier runs to completion on the query FASTA without errors
- Bootstrap support values in rdp.output fall in [0,1] and genus-rank assignments are retained only where support >= the length-appropriate cutoff (e.g. 0.60 for 200 bp in the paper, 0.3 for v5.1.0)
- Leave-one-out / cross-validation reproduction should yield at least ~99% correct genus-rank assignments for queries present in the reference set at the stated cutoffs
- Comparing against top BLAST hit assignment, the RDP route should show a much lower false positive rate (~34% vs nearly 100% in the paper) while being ~19 times faster
- Output columns parse without field misalignment (v5.1.0 specifically fixed comma-induced column misalignment in result fields)

## Limitations

- Reference database coverage is incomplete: only 22% of extant Insecta genera (16,285/72,618) have full-length species-rank CO1 barcodes in GenBank, so sequences whose congeners are missing cannot be classified (false negatives), a concern in quarantine situations
- The method has a high false negative rate in practice; lenient (90%) cutoff tables are provided for users who prefer fewer unassigned queries
- No web portal is available; the method is CLI-only
- CO1 lacks a curated community-maintained reference database equivalent to SILVA, Greengenes, or UNITE, so training set currency depends on periodic release updates

## Evidence

- [methods] These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier: "These training sets can be downloaded from https://github.com/terrimporter/CO1Classifier"
- [results] The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2.: "The 70% bootstrap support cutoff value was selected for full length (500 bp+) CO1 sequences as shown in Table 2."
- [results] The proportion of correct assignments of primer-anchored 200 bp sequences with and without 60% bootstrap support (Table 2) is also shown.: [evidence span withheld — anchor status: not_found]
- [readme] java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta: "java -Xmx8g -jar /path/to/rdp_classifier_2.13/classifier.jar -t /path/to/mydata_trained/rRNAClassifier.properties -o rdp.output query.fasta"
- [readme] As of version 5.0.0, reference sequences formatted for the RDP classifier, BLAST, and SINTAX are also available.: "As of version 5.0.0, reference sequences formatted for the RDP classifier, BLAST, and SINTAX are also available."
