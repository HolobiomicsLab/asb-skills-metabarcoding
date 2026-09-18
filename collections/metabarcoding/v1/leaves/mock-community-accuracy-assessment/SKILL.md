---
name: mock-community-accuracy-assessment
description: Use when you must compare how taxonomic classifiers (e.g., vsearch-SINTAX,
  q2-feature-classifier BLAST+LCA, VSEARCH+LCA, Naive Bayes, BOLD API) differ in completeness
  and accuracy on metabarcoding amplicons.
when_to_use_negative:
- Input sequences lack a mock community with known expected taxa, so TAR/TDR cannot
  be computed against ground truth
- The goal is to benchmark denoising or count-filtering choices rather than taxonomic
  classification completeness and accuracy
- The amplicon is not a COI animal marker, since the reference databases and the BOLD
  Classification API evaluated here are COI/arthropod-specific
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0091
tools:
- name: q2-feature-classifier
  role: Provides classify-consensus-blast and classify-consensus-vsearch used to assign
    mock and guano sequences with LCA consensus at controlled identity and query coverage
- name: VSEARCH
  role: Supplies the v2.13.4 SINTAX kmer classifier and global-alignment search backend
    used for classification comparisons
  repo: https://github.com/torognes/vsearch
- name: BOLD Classification API
  role: API-based classifier queried in batches through boldAPI_classification.R using
    the bold R package; evaluated for underclassification relative to local classifiers
  repo: https://github.com/ropensci/bold
- name: tidybug
  role: Project repository hosting the tidybug COI database files, R analysis scripts
    (boldAPI_classification.R, figure6/figure7/S4 scripts), and documentation
  repo: https://github.com/devonorourke/tidybug
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:93a2671d41b407ae5b384b51941f23524d1239660337ea9bb483169dcec5b2b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - BOLD Classification API
  - VSEARCH
  - q2-feature-classifier
  - tidybug
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 8
---

# mock-community-accuracy-assessment

## Summary

Benchmark taxonomic classifiers by running a known-composition COI mock community (CFMR_insect_mock4) and real bat guano ASVs through alignment-, kmer-, and API-based classifiers, then scoring each with Taxonomic Accuracy Rate (TAR) and Taxonomic Discovery Rate (TDR) per rank.

## When to use

Use when you must compare how taxonomic classifiers (e.g., vsearch-SINTAX, q2-feature-classifier BLAST+LCA, VSEARCH+LCA, Naive Bayes, BOLD API) differ in completeness and accuracy on metabarcoding amplicons.

## When NOT to use

- Input sequences lack a mock community with known expected taxa, so TAR/TDR cannot be computed against ground truth
- The goal is to benchmark denoising or count-filtering choices rather than taxonomic classification completeness and accuracy
- The amplicon is not a COI animal marker, since the reference databases and the BOLD Classification API evaluated here are COI/arthropod-specific

## Inputs

- Mock community amplicon sequences (CFMR_insect_mock4_wtax_noIM52) with expected taxa composition
- DADA2-filtered guano ASV table (13,407 unique sequences, basic filtering)
- COI reference database (boldCOI.derep.seqs.qza / boldCOI.derep.tax.qza; tmp_forSINTAX.fasta.gz)
- Trained Naive Bayes classifier (nbClassifer_allboldDB.qza)

## Outputs

- Per-classifier taxonomy assignments at 95%/97%/99% identity (or 0.3-0.9 confidence) for mock and guano sequences
- TAR and TDR values per classifier from class through species rank
- Tallies of common and distinct taxonomic names assigned per rank for guano ASVs
- Comparison figures generated via figure6_classifier_MockComps.R, figure7_classifier_GuanoComparisons.R, and figureS4_classifier_Consequence.R

## How to apply

Classify the mock community sequences against the tidybug COI database (boldCOI.derep.seqs.qza / boldCOI.derep.tax.qza) with classify-consensus-blast and classify-consensus-vsearch using --p-maxaccepts 1000, --p-query-cov 0.89, and percent identities of 0.95, 0.97, and 0.99; additionally run Naive Bayes (scikit-learn, confidence 0.3-0.9) and VSEARCH v2.13.4 SINTAX (--sintax_cutoff 0.3-0.9). Query the BOLD Classification API via the bold R package (boldAPI_classification.R), retaining hits above 95%, 97%, or 99% identity and applying an LCA to remaining hits, mirroring the alignment-based parameters. Apply the same classifiers (97% identity, 70% confidence threshold) to the 13,407 DADA2-filtered guano ASVs to test behavior on real data. Compute TAR = true positive / (true positive + false positive) and TDR = true positive / (true positive + false negative) for each classifier from class through species rank against the expected mock taxa, and tally common versus distinct names per rank for guano ASVs. Investigate ASVs where all non-BOLD classifiers assign a species but BOLD does not, checking whether the missing species appears in other BOLD-classified ASVs to separate classifier failure from database composition effects.

## Related tools

- **q2-feature-classifier** (Provides classify-consensus-blast and classify-consensus-vsearch used to assign mock and guano sequences with LCA consensus at controlled identity and query coverage)
- **VSEARCH** (Supplies the v2.13.4 SINTAX kmer classifier and global-alignment search backend used for classification comparisons) — https://github.com/torognes/vsearch
- **BOLD Classification API** (API-based classifier queried in batches through boldAPI_classification.R using the bold R package; evaluated for underclassification relative to local classifiers) — https://github.com/ropensci/bold
- **tidybug** (Project repository hosting the tidybug COI database files, R analysis scripts (boldAPI_classification.R, figure6/figure7/S4 scripts), and documentation) — https://github.com/devonorourke/tidybug

## Examples

```
qiime feature-classifier classify-consensus-vsearch --i-query mock_seqs.qza --i-reference-reads boldCOI.derep.seqs.qza --i-reference-taxonomy boldCOI.derep.tax.qza --p-maxaccepts 1000 --p-query-cov 0.89 --p-perc-identity 0.97 --o-classification mock_vsearch97.qza
```

## Evaluation signals

- The BOLD API reports fewer assigned names at every taxonomic rank (order through species) than all other classifiers on both mock community and guano data
- TAR and TDR values computed per rank match the reported classifier benchmark, with TAR = TP/(TP+FP) and TDR = TP/(TP+FN) against the expected mock taxa
- Alignment-based classifiers were run with the documented parameters: --p-maxaccepts 1000, --p-query-cov 0.89, identities of 0.95/0.97/0.99, and 97% identity with 70% confidence for guano
- Naive Bayes and SINTAX sweeps cover confidence/bootstrap cutoffs 0.3 through 0.9
- ASVs where all non-BOLD classifiers assign a species but BOLD does not are checked against other BOLD-classified ASVs to distinguish classifier failure from database composition issues

## Limitations

- The BOLD Classification API does not document its classification parameters or the content of its reference database, so the cause of missing assignments cannot be fully attributed
- It is unclear whether the tidybug database contents were identical to the references the BOLD API used
- The BOLD API output does not provide query coverage, so an LCA process was applied to remaining hits instead
- BOLD discontinued packaged versioned releases in 2015, hindering reproducible database comparison

## Evidence

- [abstract] the commonly used Barcode of Life Database (BOLD) Classification API assigned fewer names to samples from order through species levels using both a mock community and bat guano samples: "the commonly used Barcode of Life Database (BOLD) Classification API assigned fewer names to samples from order through species levels using both a mock community and bat guano samples"
- [abstract] biological mock communities offer a useful standard to evaluate the myriad computational decisions impacting animal metabarcoding accuracy: "biological mock communities offer a useful standard to evaluate the myriad computational decisions impacting animal metabarcoding accuracy"
- [intro] Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018): "Mock communities can provide an empirically derived filtering strategy and assess the likelihood and relative abundances of unexpected sequences (Palmer et al., 2018)"
- [results] Because their API uses a closed source reference database and classification parameters are undefined, it is unclear exactly what proportion of these missing assignments: "Because their API uses a closed source reference database and classification parameters are undefined, it is unclear exactly what proportion of these missing assignments"
- [methods] which does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used for it's classification: "which does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used for it's classification"
