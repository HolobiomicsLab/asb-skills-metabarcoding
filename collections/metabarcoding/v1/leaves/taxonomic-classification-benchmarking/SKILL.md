---
name: taxonomic-classification-benchmarking
description: Use when evaluating whether environment-specific taxonomic weights improve
  naive Bayes amplicon classification — e.g. comparing species-level bespoke accuracy
  vs. genus-level uniform accuracy on Greengenes 13_8 99% OTU references or EMPO 3
  habitat test samples.
when_to_use_negative:
- The input is a feature table or OTU abundance data that has already been classified
  — this skill benchmarks the classification step itself, not downstream community
  analyses
- Your sample type lacks sufficient Qiita/EMP characterization for bespoke weight
  assembly and you simply need any classification — use uniform or averaged global
  weights instead of running this benchmark
- The reference database or habitat differs fundamentally from Greengenes/EMPO 3 types,
  making readytowear weights and published error-rate expectations inapplicable
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: q2-feature-classifier
  role: classify-sklearn runs the naive Bayes classification under both bespoke and
    uniform weight conditions against the Greengenes 13_8 99% OTU reference
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: q2-clawback
  role: assembles the environment-specific (Qiita-derived) taxonomic class weights
    used as the bespoke condition in the benchmark
  repo: https://github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: framework hosting the classification and dissimilarity computations, including
    the Bray-Curtis metric
- name: scikit-learn
  role: provides the naive Bayes classifier implementation and statistical utilities
    underlying the benchmark
- name: imbalanced-learn
  role: supports the class-weighting machinery that bespoke taxonomic weights plug
    into
- name: readytowear
  role: source of ready-made taxonomic weights for common EMPO 3 habitats used directly
    as the bespoke condition
  repo: https://github.com/BenKaehler/readytowear
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:f29d1c7851ebf3205af90339ed7a088a74c1c15bb1452cfe7394f9531882d48a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/qiime2/q2-feature-classifier
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - readytowear
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# taxonomic-classification-benchmarking

## Summary

Benchmark bespoke (habitat-specific) taxonomic weights against uniform weights for marker-gene classification, comparing species-level bespoke error rates to genus-level uniform error rates with Bray-Curtis dissimilarity and paired t-tests. Grounded in Kaehler et al. 2019 (Nature Communications) and the q2-clawback/readytowear workflow.

## When to use

Use when evaluating whether environment-specific taxonomic weights improve naive Bayes amplicon classification — e.g. comparing species-level bespoke accuracy vs. genus-level uniform accuracy on Greengenes 13_8 99% OTU references or EMPO 3 habitat test samples.

## When NOT to use

- The input is a feature table or OTU abundance data that has already been classified — this skill benchmarks the classification step itself, not downstream community analyses
- Your sample type lacks sufficient Qiita/EMP characterization for bespoke weight assembly and you simply need any classification — use uniform or averaged global weights instead of running this benchmark
- The reference database or habitat differs fundamentally from Greengenes/EMPO 3 types, making readytowear weights and published error-rate expectations inapplicable

## Inputs

- Pre-fitted Greengenes-trained naive Bayes classifier (q2-feature-classifier artifact)
- Bespoke class-weights artifacts per test environment (from BenKaehler/readytowear)
- Test amplicon sequences with known ground-truth taxonomies (e.g. EMPO 3 habitat samples)
- Greengenes 13_8 99% OTU reference taxonomy and sequences
- Predicted and true community feature tables (BIOM) for Bray-Curtis dissimilarity

## Outputs

- fig1_comparison.csv with mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values per condition
- Per-sample classification error rates (species-level bespoke vs genus-level uniform)
- Per-sample Bray-Curtis dissimilarity between predicted and true community compositions

## How to apply

Load the pre-fitted Greengenes-trained naive Bayes classifier from q2-feature-classifier and bespoke class weights for each test environment from the readytowear repository. Classify test amplicon sequences under two conditions — species-level with bespoke (Qiita-derived) weights and genus-level with uniform weights — using q2-feature-classifier's classify-sklearn against the Greengenes 13_8 99% OTU reference. Compute per-sample error rates by comparing predicted taxonomies against ground truth, and Bray-Curtis dissimilarity between predicted and true community compositions per condition. Run a single-sided paired t-test comparing species-level bespoke error rates against genus-level uniform error rates across matched samples (likewise for Bray-Curtis dissimilarity), then compile mean error rates, mean dissimilarities, t-statistics, and P-values (expected: ~14% bespoke species vs ~16% uniform genus; Bray-Curtis 0.13 vs 0.15; P = 0.013 per Fig. 1).

## Related tools

- **q2-feature-classifier** (classify-sklearn runs the naive Bayes classification under both bespoke and uniform weight conditions against the Greengenes 13_8 99% OTU reference) — https://github.com/qiime2/q2-feature-classifier/
- **q2-clawback** (assembles the environment-specific (Qiita-derived) taxonomic class weights used as the bespoke condition in the benchmark) — https://github.com/BenKaehler/q2-clawback
- **QIIME 2** (framework hosting the classification and dissimilarity computations, including the Bray-Curtis metric)
- **scikit-learn** (provides the naive Bayes classifier implementation and statistical utilities underlying the benchmark)
- **imbalanced-learn** (supports the class-weighting machinery that bespoke taxonomic weights plug into)
- **readytowear** (source of ready-made taxonomic weights for common EMPO 3 habitats used directly as the bespoke condition) — https://github.com/BenKaehler/readytowear

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier gg-13-8-99-nb-classifier.qza --i-reads test-seqs.qza --i-class-weight bespoke-weights.qza --o-classification bespoke-preds.qza
```

## Evaluation signals

- Mean species-level bespoke error rate ≈ 14% and genus-level uniform error rate ≈ 16% across the 14 EMPO 3 habitat types
- Mean Bray-Curtis dissimilarity ≈ 0.13 (bespoke species) vs 0.15 (uniform genus)
- Single-sided paired t-test comparing bespoke vs uniform error rates yields P = 0.013, matching the paper's Fig. 1
- fig1_comparison.csv contains mean error rates, mean Bray-Curtis dissimilarities, t-statistics, and P-values for both conditions
- Predicted taxonomies must be compared against known ground-truth taxonomies for matched test samples (paired design preserved)

## Limitations

- Classification accuracy of naive Bayes classifiers degrades linearly with the degree to which the equal-abundance assumption is violated, and bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation)
- Empirical species distributions introduce error sources such as misannotated sample types and sequencing biases propagated into the taxonomic weights
- Some degradation in performance for rare species was observed under bespoke weighting
- For natural sample types lacking sufficient characterization, bespoke weights cannot be assembled — average weights estimated from global distributions are superior to uniform weights

## Evidence

- [abstract] species vs genus error rates: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [abstract] environment-specific accuracy: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [methods] ready-made weights: "If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback"
- [readme] readymade habitat classifiers: [evidence span withheld — anchor status: external_doc]
- [readme] readytowear repo: [evidence span withheld — anchor status: external_doc]
