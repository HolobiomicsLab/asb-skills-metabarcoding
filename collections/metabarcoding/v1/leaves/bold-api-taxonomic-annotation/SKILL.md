---
name: bold-api-taxonomic-annotation
description: Use when assigning taxonomy to COI amplicon sequences (e.g., bat guano
  diet ASVs) against the Barcode of Life Database, or when benchmarking the BOLD API's
  completeness against alignment-, kmer-, or Naive Bayes-based classifiers.
when_to_use_negative:
- When the goal is a single best-practice classification pipeline rather than a comparative
  benchmark — the article's stated aim is not to present a single best pathway but
  to illustrate how parameter choices affect results
- When query sequences fall outside arthropod COI scope or require a fully documented,
  versioned reference database — BOLD discontinued packaged versioned releases in
  2015 and the API's database content and parameters are undocumented
- When the input is raw reads rather than denoised representative sequences — classification
  applies to DADA2-filtered ASVs, not FASTQ reads
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0632
tools:
- name: BOLD Classification API
  role: Primary classifier being applied and benchmarked; queried in batches through
    boldAPI_classification.R with an LCA applied to hits above 95/97/99% identity
- name: bold (R package)
  role: R interface used to access the BOLD database and submit API classification
    queries, following its vignette
  repo: https://github.com/ropensci/bold
- name: q2-feature-classifier
  role: Comparison classifier using classify-consensus-blast and classify-consensus-vsearch
    (--p-maxaccepts 1000, --p-query-cov 0.89) at 0.95–0.99 identities
- name: vsearch-SINTAX
  role: Kmer-based comparison classifier (VSEARCH v2.13.4) run with --sintax_cutoff
    0.3–0.9 against tmp_forSINTAX.fasta.gz
  repo: https://github.com/torognes/vsearch
- name: Naive Bayes (q2 scikit-learn)
  role: Comparison classifier trained as nbClassifer_allboldDB.qza and applied at
    confidence thresholds 0.3, 0.5, 0.7, 0.8, 0.9
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:432caa2371974f9e901f32b206c326744e37a6b83154b5bc7cb02a7bd5766f7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ropensci/bold
  tools:
  - BOLD Classification API
  - Naive Bayes (q2 scikit-learn)
  - bold (R package)
  - q2-feature-classifier
  - vsearch-SINTAX
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.16666666666666666
n_claims: 6
---

# BOLD API taxonomic annotation

## Summary

Classify COI metabarcoding sequences (ASVs) using the BOLD Classification API via the R 'bold' package, retaining hits above a percent-identity threshold and applying a lowest common ancestor (LCA) to remaining hits, then benchmark the resulting assignments against other classifiers using TAR/TDR metrics.

## When to use

Use when assigning taxonomy to COI amplicon sequences (e.g., bat guano diet ASVs) against the Barcode of Life Database, or when benchmarking the BOLD API's completeness against alignment-, kmer-, or Naive Bayes-based classifiers.

## When NOT to use

- When the goal is a single best-practice classification pipeline rather than a comparative benchmark — the article's stated aim is not to present a single best pathway but to illustrate how parameter choices affect results
- When query sequences fall outside arthropod COI scope or require a fully documented, versioned reference database — BOLD discontinued packaged versioned releases in 2015 and the API's database content and parameters are undocumented
- When the input is raw reads rather than denoised representative sequences — classification applies to DADA2-filtered ASVs, not FASTQ reads

## Inputs

- DADA2-filtered ASV sequences (e.g., 13,407 unique COI sequences from bat guano samples after basic filtering)
- Mock community sequences (CFMR_insect_mock4_wtax_noIM52) with known expected taxa
- Tidybug BOLD COI reference database (boldCOI.derep.seqs.qza / boldCOI.derep.tax.qza)
- BOLD Classification API access via the bold R package

## Outputs

- Per-ASV taxonomic assignments from the BOLD API at order through species ranks
- Per-rank counts of common and distinct taxonomic names across classifiers for guano ASVs
- TAR and TDR values per classifier per rank for the mock community
- Comparison figures (figure6_classifier_MockComps.R, figure7_classifier_GuanoComparisons.R, figureS4_classifier_Consequence.R)

## How to apply

Query DADA2-filtered ASVs (e.g., 13,407 unique guano sequences from basic filtering) to the BOLD Classification API in batches using the bold R package (boldAPI_classification.R), retaining only hits above the chosen identity threshold (95%, 97%, or 99% to mirror the alignment-based classifiers). Because the API does not report query coverage, apply an LCA process to the remaining hits to produce a consensus assignment. Run the same query set through the competing classifiers—q2-feature-classifier's classify-consensus-blast and classify-consensus-vsearch (--p-maxaccepts 1000, --p-query-cov 0.89, percent identities 0.95/0.97/0.99), a Naive Bayes scikit-learn classifier (nbClassifer_allboldDB.qza) at confidence 0.3–0.9, and vsearch-SINTAX (--sintax_cutoff 0.3–0.9)—against the tidybug boldCOI reference database. Compare named-taxon counts per rank (order through species) and compute Taxonomic Accuracy Rate (TAR = TP/(TP+FP)) and Taxonomic Discovery Rate (TDR = TP/(TP+FN)) against expected mock taxa, processing outputs with figure6_classifier_MockComps.R and figure7_classifier_GuanoComparisons.R.

## Related tools

- **BOLD Classification API** (Primary classifier being applied and benchmarked; queried in batches through boldAPI_classification.R with an LCA applied to hits above 95/97/99% identity)
- **bold (R package)** (R interface used to access the BOLD database and submit API classification queries, following its vignette) — https://github.com/ropensci/bold
- **q2-feature-classifier** (Comparison classifier using classify-consensus-blast and classify-consensus-vsearch (--p-maxaccepts 1000, --p-query-cov 0.89) at 0.95–0.99 identities)
- **vsearch-SINTAX** (Kmer-based comparison classifier (VSEARCH v2.13.4) run with --sintax_cutoff 0.3–0.9 against tmp_forSINTAX.fasta.gz) — https://github.com/torognes/vsearch
- **Naive Bayes (q2 scikit-learn)** (Comparison classifier trained as nbClassifer_allboldDB.qza and applied at confidence thresholds 0.3, 0.5, 0.7, 0.8, 0.9)

## Examples

```
Rscript boldAPI_classification.R --input guano_asvs.fasta --identity 0.97 --lca
```

## Evaluation signals

- The BOLD API should report the lowest number of named taxa at every taxonomic rank (order through species) relative to vsearch-SINTAX, BLAST+LCA, VSEARCH+LCA, and Naive Bayes on both the mock community and bat guano samples
- TAR/TDR values computed against expected mock taxa should match the reported classifier benchmark when TAR = TP/(TP+FP) and TDR = TP/(TP+FN)
- For ASVs where all non-BOLD classifiers assigned a species but BOLD did not, check whether the missing species appears in other BOLD-classified ASVs to distinguish classifier failure from database composition issues
- LCA is applied only to hits retained above the chosen identity cutoff (95%, 97%, or 99%) since the API output provides no query coverage parameter
- Batched API queries through boldAPI_classification.R should complete for all 13,407 guano ASVs without dropped sequences

## Limitations

- BOLD API uses a closed-source reference database with undefined classification parameters, making it unclear exactly what drives missing assignments
- It is unclear whether the tidybug database contents match those references the BOLD API used
- BOLD API output does not provide query coverage, so an LCA must be applied to remaining hits
- BOLD discontinued packaged versioned releases in 2015, undermining reproducibility of API classifications
- No attempt was made to determine the prevalence of BOLD records lacking Phylum/Class information

## Evidence

- [abstract] the commonly used Barcode of Life Database (BOLD) Classification API assigned fewer names to samples from order through species levels using both a mock community and bat guano samples: "the commonly used Barcode of Life Database (BOLD) Classification API assigned fewer names to samples from order through species levels using both a mock community and bat guano samples"
- [methods] which does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used for it's classification: "which does not provide details about specific parameters invoked during classification, nor does it specify the content of the database used for it's classification"
- [methods] Because query coverage was not a parameter that was provided in the output, we applied an LCA process to those remaining hits: "Because query coverage was not a parameter that was provided in the output, we applied an LCA process to those remaining hits"
- [methods] We followed the vignette provided by the bold R package to download data from the Barcode of Life Database: "We followed the vignette provided by the [bold R package](https://github.com/ropensci/bold) to download data from the [Barcode of Life Database]"
- [discussion] While BOLD had previously presented packaged versioned releases, this practice ended in 2015.: "While BOLD had previously presented packaged versioned releases, this practice ended in 2015."
