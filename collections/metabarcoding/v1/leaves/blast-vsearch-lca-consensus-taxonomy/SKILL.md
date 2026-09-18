---
name: blast-vsearch-lca-consensus-taxonomy
description: Use when benchmarking taxonomic classifiers on COI ASVs (e.g., mock communities
  or diet samples) to quantify which classifiers assign the most names from class
  through species rank and how accuracy trades off against completeness.
when_to_use_negative:
- When the input is already a classified feature table or taxonomy artifact — this
  skill benchmarks raw ASV classification, not post-hoc taxonomy comparison.
- When the goal is to evaluate denoising (DADA2 vs Deblur) or count transformation
  (RA vs PA) decisions rather than classifier choice — those are separate benchmarking
  questions in the same study.
- When a single best classifier pathway is sought; the authors state their 'aim is
  not to present a single best pathway for all animal metabarcoding projects, but
  to illustrate how these processes are affected by program or parameter choices.'
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: q2-feature-classifier
  role: Runs classify-consensus-blast (BLAST+LCA) and classify-consensus-vsearch (VSEARCH+LCA)
    with maxaccepts 1000, query-cov 0.89, and 95/97/99% identities against the tidybug
    BOLD COI database
  repo: https://github.com/devonorourke/tidybug
- name: BLAST
  role: Local alignment engine behind classify-consensus-blast; the article notes
    'one may choose to classify sequences using local alignment with software like
    BLAST'
- name: VSEARCH
  role: Global aligner behind classify-consensus-vsearch and the vsearch-SINTAX kmer
    classifier (v2.13.4); mirrors parameters from the VSEARCH Wiki pipeline
  repo: https://github.com/torognes/vsearch
- name: vsearch-SINTAX
  role: Kmer-based classifier run with --sintax_cutoff 0.3–0.9 against tmp_forSINTAX.fasta.gz
    as a comparator
  repo: https://github.com/torognes/vsearch
- name: BOLD Classification API
  role: Comparator classifier invoked via the bold R package through boldAPI_classification.R;
    assignments are underclassified relative to all other classifiers
  repo: https://github.com/ropensci/bold
- name: Naive Bayes (q2 scikit-learn)
  role: Trained classifier nbClassifer_allboldDB.qza evaluated at confidence thresholds
    0.3, 0.5, 0.7, 0.8, 0.9
  repo: https://github.com/devonorourke/tidybug
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:22520eadd24ebc4d666e6a1ddb298f65a59572c4023d252849035f8c9809119e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/devonorourke/tidybug
  tools:
  - BLAST
  - BOLD Classification API
  - Naive Bayes (q2 scikit-learn)
  - VSEARCH
  - q2-feature-classifier
  - vsearch-SINTAX
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# blast-vsearch-lca-consensus-taxonomy

## Summary

Benchmark taxonomy assignment of COI metabarcoding ASVs using q2-feature-classifier's BLAST+LCA and VSEARCH+LCA consensus classifiers against the tidybug BOLD-derived COI database, alongside vsearch-SINTAX, Naive Bayes, and the BOLD Classification API, to measure relative completeness (TAR/TDR) and classifier underclassification bias.

## When to use

Use when benchmarking taxonomic classifiers on COI ASVs (e.g., mock communities or diet samples) to quantify which classifiers assign the most names from class through species rank and how accuracy trades off against completeness.

## When NOT to use

- When the input is already a classified feature table or taxonomy artifact — this skill benchmarks raw ASV classification, not post-hoc taxonomy comparison.
- When the goal is to evaluate denoising (DADA2 vs Deblur) or count transformation (RA vs PA) decisions rather than classifier choice — those are separate benchmarking questions in the same study.
- When a single best classifier pathway is sought; the authors state their 'aim is not to present a single best pathway for all animal metabarcoding projects, but to illustrate how these processes are affected by program or parameter choices.'

## Inputs

- Mock community sequences (CFMR_insect_mock4_wtax_noIM52)
- tidybug reference database: boldCOI.derep.seqs.qza and boldCOI.derep.tax.qza
- Naive Bayes trained classifier: nbClassifer_allboldDB.qza
- SINTAX-formatted reference FASTA: tmp_forSINTAX.fasta.gz
- DADA2-filtered bat guano ASVs (13,407 unique sequences, basic filtering)
- Expected mock community taxa list for TAR/TDR scoring

## Outputs

- Per-classifier taxonomy assignments (qza / SINTAX tab / BOLD API results) at class through species rank
- TAR and TDR values per classifier per taxonomic rank
- Counts of common and distinct taxonomic names assigned per rank across classifiers (guano ASVs)
- Comparison figures from figure6_classifier_MockComps.R, figure7_classifier_GuanoComparisons.R, figureS4_classifier_Consequence.R

## How to apply

Classify mock community sequences (CFMR_insect_mock4_wtax_noIM52) against the tidybug database (boldCOI.derep.seqs.qza / boldCOI.derep.tax.qza) with classify-consensus-blast and classify-consensus-vsearch using --p-maxaccepts 1000, --p-query-cov 0.89, and percent identities of 0.95, 0.97, and 0.99. Also run the Naive Bayes scikit-learn classifier (nbClassifer_allboldDB.qza) at confidence thresholds 0.3–0.9 and vsearch-SINTAX (v2.13.4, --sintax_cutoff 0.3–0.9) against tmp_forSINTAX.fasta.gz, then apply the BOLD API via the bold R package, retaining hits above 95/97/99% identity and applying an LCA to remaining hits (the API provides no query coverage parameter). For the guano samples, classify the 13,407 DADA2-filtered ASVs with all classifiers at a 97% identity and 70% confidence threshold, batching BOLD API queries through boldAPI_classification.R. Compute Taxonomic Accuracy Rate (TAR = true positive / (true positive + false positive)) and Taxonomic Discovery Rate (TDR = true positive / (true positive + false negative)) per rank against the expected mock taxa, and tally common vs distinct assigned names per rank for the guano ASVs. Generate comparisons with figure6_classifier_MockComps.R, figure7_classifier_GuanoComparisons.R, and figureS4_classifier_Consequence.R; investigate ASVs where all non-BOLD classifiers assigned a species but BOLD did not to distinguish classifier failure from database composition issues.

## Related tools

- **q2-feature-classifier** (Runs classify-consensus-blast (BLAST+LCA) and classify-consensus-vsearch (VSEARCH+LCA) with maxaccepts 1000, query-cov 0.89, and 95/97/99% identities against the tidybug BOLD COI database) — https://github.com/devonorourke/tidybug
- **BLAST** (Local alignment engine behind classify-consensus-blast; the article notes 'one may choose to classify sequences using local alignment with software like BLAST')
- **VSEARCH** (Global aligner behind classify-consensus-vsearch and the vsearch-SINTAX kmer classifier (v2.13.4); mirrors parameters from the VSEARCH Wiki pipeline) — https://github.com/torognes/vsearch
- **vsearch-SINTAX** (Kmer-based classifier run with --sintax_cutoff 0.3–0.9 against tmp_forSINTAX.fasta.gz as a comparator) — https://github.com/torognes/vsearch
- **BOLD Classification API** (Comparator classifier invoked via the bold R package through boldAPI_classification.R; assignments are underclassified relative to all other classifiers) — https://github.com/ropensci/bold
- **Naive Bayes (q2 scikit-learn)** (Trained classifier nbClassifer_allboldDB.qza evaluated at confidence thresholds 0.3, 0.5, 0.7, 0.8, 0.9) — https://github.com/devonorourke/tidybug

## Examples

```
qiime feature-classifier classify-consensus-vsearch --i-query mock_seqs.qza --i-reference-reads boldCOI.derep.seqs.qza --i-reference-taxonomy boldCOI.derep.tax.qza --p-maxaccepts 1000 --p-query-cov 0.89 --p-perc-identity 0.97 --o-classification vsearch_lca_tax.qza
```

## Evaluation signals

- The BOLD API should show fewer assigned names at order through species levels than every other classifier on both the mock community and the bat guano samples.
- TAR and TDR values per classifier per rank should match the reported classifier benchmark (TAR = TP/(TP+FP); TDR = TP/(TP+FN)).
- Mock classification should use all three percent identities (0.95, 0.97, 0.99) with --p-maxaccepts 1000 and --p-query-cov 0.89 for the alignment-based classifiers.
- Guano classification should use a 97% identity threshold and 70% confidence threshold across the four QIIME 2/VSEARCH-based classifiers, and BOLD queries should be batched through boldAPI_classification.R.
- ASVs where all non-BOLD classifiers assigned a species but BOLD did not should be checked for whether the missing species appears in other BOLD-classified ASVs, to separate classifier failure from database composition gaps.

## Limitations

- The BOLD Classification API 'does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used for its classification' — missing assignments cannot be attributed precisely.
- It is unclear whether the tidybug database contents match those used by the BOLD API internally.
- BOLD API output does not provide query coverage, so an LCA process was applied to remaining hits instead.
- BOLD discontinued packaged versioned releases in 2015, and no single curated and versioned reference database resource is widely used in animal diet studies, making cross-study classifier comparisons difficult.
- No attempt was made to determine the prevalence of BOLD records lacking Phylum/Class information.

## Evidence

- [abstract] the commonly used Barcode of Life Database (BOLD) Classification API assigned fewer names to samples from order through species levels: "the commonly used Barcode of Life Database (BOLD) Classification API assigned fewer names to samples from order through species levels"
- [abstract] compared to all other classifiers (vsearch‐SINTAX and q2‐feature-classifier's BLAST + LCA, VSEARCH + LCA, and Naive Bayes classifiers): [evidence span withheld — anchor status: not_found]
- [methods] which does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used: "which does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used for it's classification"
- [methods] Because query coverage was not a parameter that was provided in the output, we applied an LCA process to those remaining hits: "Because query coverage was not a parameter that was provided in the output, we applied an LCA process to those remaining hits"
- [intro] We benchmarked taxonomic classification using both a mock community and bat guano samples to evaluate relative classifier completeness and accuracy: "We benchmarked taxonomic classification using both a mock community and bat guano samples to evaluate relative classifier completeness and accuracy"
