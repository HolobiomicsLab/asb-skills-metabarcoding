---
name: taxonomic-classifier-benchmarking
description: Use when a COI metabarcoding dataset needs taxonomic classification and
  you must decide which classifier (alignment, kmer, or web API based) assigns the
  most complete and accurate names against a curated reference database.
when_to_use_negative:
- Input is already a classified feature table or taxonomy artifact — this skill evaluates
  classification choices, not post-hoc taxonomy reconciliation
- The marker gene or reference database is non-COI / non-animal (e.g. 16S with SILVA/Greengenes/UNITE)
  — the benchmark findings are specific to COI with BOLD- and GenBank-derived references
- You need a single best-practice pipeline rather than a comparison — the article's
  stated aim is to illustrate how processes are affected by program or parameter choices,
  not to present one best pathway
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: q2-feature-classifier
  role: Runs classify-consensus-blast and classify-consensus-vsearch (BLAST+LCA, VSEARCH+LCA)
    and the Naive Bayes scikit-learn classifier against the tidybug BOLD reference
  repo: https://github.com/devonorourke/tidybug
- name: vsearch-SINTAX
  role: VSEARCH v2.13.4 SINTAX kmer-based classifier (--sintax_cutoff 0.3-0.9) against
    tmp_forSINTAX.fasta.gz
  repo: https://github.com/torognes/vsearch
- name: BOLD Classification API
  role: Web classifier queried via the bold R package (boldAPI_classification.R) with
    an LCA applied to hits above 95/97/99% identity
  repo: https://github.com/ropensci/bold
- name: BLAST
  role: Local-alignment engine underlying q2-feature-classifier's classify-consensus-blast
    assignments
- name: VSEARCH
  role: Global aligner powering classify-consensus-vsearch and the v2.13.4 SINTAX
    implementation
  repo: https://github.com/torognes/vsearch
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:f5bff5ae7e1ff2e34929c06230cb8f573712db6a84635dedd6d05c25dba2f4cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/devonorourke/tidybug
  tools:
  - BLAST
  - BOLD Classification API
  - VSEARCH
  - q2-feature-classifier
  - vsearch-SINTAX
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# taxonomic-classifier-benchmarking

## Summary

Benchmark taxonomic classifiers (vsearch-SINTAX, q2-feature-classifier's BLAST+LCA and VSEARCH+LCA, Naive Bayes, and the BOLD Classification API) on a known mock community and real bat guano COI ASVs, quantifying relative completeness via Taxonomic Accuracy Rate (TAR) and Taxonomic Discovery Rate (TDR) from class through species ranks.

## When to use

Use when a COI metabarcoding dataset needs taxonomic classification and you must decide which classifier (alignment, kmer, or web API based) assigns the most complete and accurate names against a curated reference database.

## When NOT to use

- Input is already a classified feature table or taxonomy artifact — this skill evaluates classification choices, not post-hoc taxonomy reconciliation
- The marker gene or reference database is non-COI / non-animal (e.g. 16S with SILVA/Greengenes/UNITE) — the benchmark findings are specific to COI with BOLD- and GenBank-derived references
- You need a single best-practice pipeline rather than a comparison — the article's stated aim is to illustrate how processes are affected by program or parameter choices, not to present one best pathway

## Inputs

- Mock community ASV sequences with known expected taxonomy (CFMR_insect_mock4_wtax_noIM52)
- DADA2-filtered bat guano ASVs (13,407 unique sequences, basic filtering)
- BOLD-derived reference database (boldCOI.derep.seqs.qza and boldCOI.derep.tax.qza; tmp_forSINTAX.fasta.gz for SINTAX)
- Trained Naive Bayes classifier (nbClassifer_allboldDB.qza)
- Guano and mock feature tables and representative sequences (QIIME 2 .qza artifacts)

## Outputs

- Per-classifier taxonomy assignments (.qza / tabular) at 95/97/99% identity or 0.3-0.9 confidence thresholds
- TAR and TDR values per classifier per taxonomic rank (class through species) against expected mock taxa
- Counts of common and distinct taxonomic names assigned per rank for the guano ASVs
- Comparison figures (figure6_classifier_MockComps.R, figure7_classifier_GuanoComparisons.R, figureS4_classifier_Consequence.R outputs)

## How to apply

Classify the mock community sequences (CFMR_insect_mock4_wtax_noIM52) against the tidybug BOLD database (boldCOI.derep.seqs.qza / boldCOI.derep.tax.qza) using q2-feature-classifier's classify-consensus-blast and classify-consensus-vsearch with --p-maxaccepts 1000, --p-query-cov 0.89, and percent identities of 0.95, 0.97, and 0.99; also classify with the Naive Bayes scikit-learn classifier at confidence thresholds 0.3-0.9 and vsearch-SINTAX (--sintax_cutoff 0.3-0.9). Classify the same mock sequences through the BOLD API via the bold R package, retaining hits above 95%, 97%, or 99% identity and applying an LCA to remaining hits to mirror the alignment-based parameters. Classify the DADA2-filtered guano ASVs (13,407 unique sequences under basic filtering) with all classifiers at a 97% identity / 70% confidence threshold, batching BOLD queries through boldAPI_classification.R. Compute TAR = true positive / (true positive + false positive) and TDR = true positive / (true positive + false negative) per rank against expected mock taxa, tally common and distinct assigned names per rank for guano ASVs, and render comparisons with figure6_classifier_MockComps.R, figure7_classifier_GuanoComparisons.R, and figureS4_classifier_Consequence.R.

## Related tools

- **q2-feature-classifier** (Runs classify-consensus-blast and classify-consensus-vsearch (BLAST+LCA, VSEARCH+LCA) and the Naive Bayes scikit-learn classifier against the tidybug BOLD reference) — https://github.com/devonorourke/tidybug
- **vsearch-SINTAX** (VSEARCH v2.13.4 SINTAX kmer-based classifier (--sintax_cutoff 0.3-0.9) against tmp_forSINTAX.fasta.gz) — https://github.com/torognes/vsearch
- **BOLD Classification API** (Web classifier queried via the bold R package (boldAPI_classification.R) with an LCA applied to hits above 95/97/99% identity) — https://github.com/ropensci/bold
- **BLAST** (Local-alignment engine underlying q2-feature-classifier's classify-consensus-blast assignments)
- **VSEARCH** (Global aligner powering classify-consensus-vsearch and the v2.13.4 SINTAX implementation) — https://github.com/torognes/vsearch

## Examples

```
qiime feature-classifier classify-consensus-vsearch --i-query mock_asvs.qza --i-reference-reads boldCOI.derep.seqs.qza --i-reference-taxonomy boldCOI.derep.tax.qza --p-maxaccepts 1000 --p-query-cov 0.89 --p-perc-identity 0.97 --o-classification mock_vsearch97.qza
```

## Evaluation signals

- The BOLD API should report fewer named taxa at every rank (order through species) than all other classifiers on both the mock community and the bat guano samples
- TAR and TDR values per classifier and rank must be recomputable from the mock community's expected taxa table (true positives, false positives, false negatives)
- Classifier parameter grids should be reproducible: identity 0.95/0.97/0.99, confidence 0.3-0.9, --p-maxaccepts 1000, --p-query-cov 0.89
- For ASVs where all non-BOLD classifiers assign a species but BOLD does not, a check should determine whether the missing species appears in other BOLD-classified ASVs, distinguishing classifier failure from database composition issues
- Comparison figures regenerate without error from figure6_classifier_MockComps.R, figure7_classifier_GuanoComparisons.R, and figureS4_classifier_Consequence.R

## Limitations

- The BOLD Classification API is a closed-source black box: its classification parameters and the content of its database are undocumented, so it is unclear what drives its missing assignments
- It is unclear whether the tidybug database contents were identical to the references the BOLD API used, making comparisons partly confounded
- The BOLD API output does not provide query coverage, so an LCA process was applied to the remaining hits rather than filtering on coverage
- No attempt was made to determine how pervasive BOLD records lacking Phylum/Class information are, and BOLD discontinued packaged versioned releases in 2015, hampering reproducibility

## Evidence

- [abstract] BOLD underclassifies relative to other classifiers: "the commonly used Barcode of Life Database (BOLD) Classification API assigned fewer names to samples from order through species levels using both a mock community and bat guano samples"
- [methods] BOLD API parameters undocumented: "which does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used for it's classification"
- [methods] LCA applied due to missing query coverage: "Because query coverage was not a parameter that was provided in the output, we applied an LCA process to those remaining hits"
- [methods] bold R package used to access BOLD: "We accessed arthropod records from the BOLD database using the bold R package (Chamberlain, 2017) on 24 February 2019"
- [intro] Benchmark rationale: "We benchmarked taxonomic classification using both a mock community and bat guano samples to evaluate relative classifier completeness and accuracy"
- [readme] tidybug repo hosts scripts and data: "Complete details including scripts and data files used in this manuscript are available at the GitHub repo for this project: https://github.com/devonorourke/tidybug"
