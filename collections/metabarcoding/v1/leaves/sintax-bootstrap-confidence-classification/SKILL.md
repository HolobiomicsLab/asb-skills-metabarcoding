---
name: sintax-bootstrap-confidence-classification
description: Use when ASV/representative sequences (e.g. FASTA of DADA2-filtered COI
  reads) need taxonomic assignment via a kmer/bootstrap classifier to compare completeness
  and accuracy across classifiers.
when_to_use_negative:
- Input is already a classified feature table (qiime2 taxonomy artifact) — reclassification
  adds nothing
- Query sequences are not COI/barcode amplicons; this reference (tidybug BOLD/GenBank
  COI database) is arthropod-COI specific
- You need query-coverage-aware local alignment results — SINTAX is kmer-based and
  the BOLD API route is the one the paper contrasts against it
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3697
tools:
- name: vsearch-SINTAX
  role: Performs kmer-based taxonomic classification with bootstrap confidence cutoffs
    (0.3–0.9) against the formatted BOLD COI reference
  repo: https://github.com/torognes/vsearch
- name: VSEARCH
  role: Engine running the SINTAX classifier (v2.13.4) used on mock and guano sequences
  repo: https://github.com/torognes/vsearch
- name: CO1Classifier
  role: Example of SINTAX-formatted COI reference training sets the workflow's reference
    formatting parallels
  repo: https://github.com/terrimporter/CO1Classifier
- name: q2-feature-classifier
  role: Companion classifier whose BLAST+LCA, VSEARCH+LCA, and Naive Bayes outputs
    SINTAX is benchmarked against
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:a3befcb968836f9c18f830dab933055cf9f845b70d58f1e763b5c85aeed35924
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - BOLD Classification API
  - CO1Classifier
  - VSEARCH
  - q2-feature-classifier
  - vsearch-SINTAX
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sintax-bootstrap-confidence-classification

## Summary

Classify COI amplicon sequences with VSEARCH's kmer-based SINTAX implementation against a BOLD-derived reference, applying bootstrap support cutoffs (0.3–0.9) so assignments can be benchmarked against alignment-based and Naive Bayes classifiers by named taxa and TAR/TDR.

## When to use

Use when ASV/representative sequences (e.g. FASTA of DADA2-filtered COI reads) need taxonomic assignment via a kmer/bootstrap classifier to compare completeness and accuracy across classifiers.

## When NOT to use

- Input is already a classified feature table (qiime2 taxonomy artifact) — reclassification adds nothing
- Query sequences are not COI/barcode amplicons; this reference (tidybug BOLD/GenBank COI database) is arthropod-COI specific
- You need query-coverage-aware local alignment results — SINTAX is kmer-based and the BOLD API route is the one the paper contrasts against it

## Inputs

- Dereplicated COI reference sequences with taxonomy (boldCOI.derep.seqs.qza / boldCOI.derep.tax.qza, formatted as tmp_forSINTAX.fasta.gz)
- Mock community representative sequences (CFMR_insect_mock4_wtax_noIM52)
- DADA2-filtered bat guano ASV sequences (13,407 unique sequences, basic filtering)
- Expected mock community taxa list for TAR/TDR scoring

## Outputs

- Per-sequence SINTAX assignments with bootstrap confidences at each taxonomic rank
- Per-rank counts of named taxa per classifier for mock and guano datasets
- TAR and TDR tables (class through species rank) feeding figure6/figure7 comparison figures

## How to apply

Format the dereplicated BOLD COI reference (boldCOI.derep.seqs.qza / boldCOI.derep.tax.qza, exported as tmp_forSINTAX.fasta.gz) so lineage labels are SINTAX-compliant (k; p; c; o; f; g; s; prefixed). Run VSEARCH v2.13.4's SINTAX implementation on the mock community sequences (CFMR_insect_mock4_wtax_noIM52) and the 13,407 DADA2-filtered guano ASVs, sweeping --sintax_cutoff from 0.3 to 0.9 to mirror the confidence thresholds tested for the q2 Naive Bayes classifier. Downstream, tally the number of named taxa per rank and compute Taxonomic Accuracy Rate (TAR = TP/(TP+FP)) and Taxonomic Discovery Rate (TDR = TP/(TP+FN)) against expected mock taxa using figure6_classifier_MockComps.R / figure7_classifier_GuanoComparisons.R. Rationale: the paper uses this to establish that the BOLD Classification API underclassifies relative to vsearch-SINTAX and the other classifiers from order through species levels.

## Related tools

- **vsearch-SINTAX** (Performs kmer-based taxonomic classification with bootstrap confidence cutoffs (0.3–0.9) against the formatted BOLD COI reference) — https://github.com/torognes/vsearch
- **VSEARCH** (Engine running the SINTAX classifier (v2.13.4) used on mock and guano sequences) — https://github.com/torognes/vsearch
- **CO1Classifier** (Example of SINTAX-formatted COI reference training sets the workflow's reference formatting parallels) — https://github.com/terrimporter/CO1Classifier
- **q2-feature-classifier** (Companion classifier whose BLAST+LCA, VSEARCH+LCA, and Naive Bayes outputs SINTAX is benchmarked against)

## Examples

```
vsearch --sintax mock_asvs.fa --db tmp_forSINTAX.fasta.gz --sintax_cutoff 0.3 --tabbedfile mock_sintax_c0.3.tsv
```

## Evaluation signals

- SINTAX assignments generated for both the mock community and the 13,407 guano ASVs with cutoffs swept 0.3–0.9
- TAR/TDR values for SINTAX fall within the reported classifier benchmark range (class through species rank)
- Number of named taxa per rank for SINTAX equals or exceeds the BOLD API at order through species levels on both mock and guano data
- Output per-rank name tallies reconcile with figure6_classifier_MockComps.R and figure7_classifier_GuanoComparisons.R inputs

## Limitations

- SINTAX bootstrap cutoff choice affects completeness; low cutoffs (e.g. 0.3) trade accuracy (higher FP) for discovery (higher TDR), mirroring the Naive Bayes confidence sweep
- Accuracy depends on reference database composition; the paper notes no single curated, versioned resource is widely used in animal diet studies, making cross-study comparisons difficult
- The 181 bp COI fragment is short relative to full-length barcode references, constraining achievable bootstrap support at deeper ranks

## Evidence

- [abstract] compared to all other classifiers (vsearch‐SINTAX and q2‐feature-classifier's BLAST + LCA, VSEARCH + LCA, and Naive Bayes classifiers): [evidence span withheld — anchor status: not_found]
- [intro] kmer-based classifiers such as SINTAX (Edgar, 2016) or Naive Bayes classification: [evidence span withheld — anchor status: not_found]
- [other] Classify the same mock sequences with the q2 Naive Bayes scikit-learn classifier ... and with the VSEARCH v2.13.4 SINTAX implementation (--sintax_cutoff 0.3-0.9) against tmp_forSINTAX.fasta.gz.: [evidence span withheld — anchor status: not_found]
- [readme] As of version 5.0.0, reference sequences formatted for the RDP classifier, BLAST, and SINTAX are also available.: [evidence span withheld — anchor status: not_found]
