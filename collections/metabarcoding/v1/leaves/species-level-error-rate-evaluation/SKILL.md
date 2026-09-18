---
name: species-level-error-rate-evaluation
description: Use when comparing naive Bayes taxonomic classifiers trained with environment-specific
  (bespoke) weights versus uniform weights and you need per-habitat species-level
  error rates with paired statistical testing.
when_to_use_negative:
- The sample environment is poorly characterized such that bespoke weights cannot
  be assembled — use average weights estimated from global microbial species distributions
  instead
- The query samples come from a highly unusual distribution (e.g., synthetic populations)
  without matching reference samples from which to compile bespoke weights
- Only genus-level or coarser classification accuracy is required, where uniform weights
  already perform comparably (16% genus-level error for existing classifiers)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: QIIME 2
  role: Framework in which the naive Bayes classifiers are trained and queries classified
  repo: https://qiime2.org
- name: q2-feature-classifier
  role: Provides fit-classifier-naive-bayes (MultinomialNB with class_weight='balanced')
    and classify-sklearn at confidence 0.7 for the two weighting schemes
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: q2-clawback
  role: Generates bespoke per-habitat class weights via fetch_Qiita_samples and generate_class_weights
    as the treatment condition
  repo: https://github.com/BenKaehler/q2-clawback
- name: scikit-learn
  role: Supplies the MultinomialNB classifier underlying the error-rate comparison
- name: redbiom
  role: Fetches environment-matched Qiita samples/features for the 14 EMPO 3 habitats
    used as query data
  repo: https://github.com/biocore/redbiom
- name: readytowear
  role: Source of precomputed habitat-specific class weights, avoiding a rerun of
    clawback
  repo: https://github.com/BenKaehler/readytowear
merged_aliases:
- species-level-error-rate-benchmarking
merged_alias_records:
- alias: species-level-error-rate-benchmarking
  slug: species-level-error-rate-benchmarking
  jaccard_score: 0.6667
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:5d1ade7f72af6f45a1d77165f8223523590e5a5d9a15860db4bb8a5a79139f40
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - readytowear
  - redbiom
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# species-level-error-rate-evaluation

## Summary

Benchmark taxonomic classifier performance at species level by computing per-habitat error rates for bespoke (environment-specific) versus uniform class weights across the 14 EMPO 3 habitats, then testing paired differences. This reproduces the paper's headline result that habitat-specific weights cut average species-level error from 25% to 14%.

## When to use

Use when comparing naive Bayes taxonomic classifiers trained with environment-specific (bespoke) weights versus uniform weights and you need per-habitat species-level error rates with paired statistical testing.

## When NOT to use

- The sample environment is poorly characterized such that bespoke weights cannot be assembled — use average weights estimated from global microbial species distributions instead
- The query samples come from a highly unusual distribution (e.g., synthetic populations) without matching reference samples from which to compile bespoke weights
- Only genus-level or coarser classification accuracy is required, where uniform weights already perform comparably (16% genus-level error for existing classifiers)

## Inputs

- Greengenes 99% OTU reference sequences and taxonomy (QIIME 2 feature-classifier reference data)
- EMPO 3 habitat-specific taxonomic class weights from the readytowear repository (or q2-clawback-generated weights)
- Uniform class-weight baseline (all species equally likely)
- Held-out query amplicon sequence variants per EMPO 3 habitat (e.g., fetched from Qiita via redbiom)

## Outputs

- fig1_error_rates.csv with per-habitat species-level error rates for bespoke and uniform classifiers
- supplementary_fig5_error_rates.csv with per-phylum (Proteobacteria, Firmicutes) error rates and paired t-test P-values
- Paired t-test statistics comparing uniform vs bespoke error rates (overall P = 5.8e-5)

## How to apply

Train a q2-feature-classifier fit-classifier-naive-bayes multinomial naive Bayes classifier (scikit-learn MultinomialNB, class_weight='balanced') on Greengenes 99% OTU reference sequences once per EMPO 3 habitat with bespoke weights (from readytowear or generated via q2-clawback's generate_class_weights) and once with uniform weights. Classify each habitat's held-out query amplicon sequence variants with classify-sklearn at confidence 0.7 under both weighting schemes, then compute species-level error rates per habitat. Compare the paired per-habitat error rates with a paired t-test and tabulate per-habitat and overall averages. Correct application is judged by recovering the paper's averages of 14% (bespoke) vs 25% (uniform) with paired t-test P = 5.8e-5, and per-phylum drops for Proteobacteria (35.4% to 22.3%, P ≈ 1.4e-6) and Firmicutes (43.6% to 24.3%, P ≈ 8.4e-6).

## Related tools

- **QIIME 2** (Framework in which the naive Bayes classifiers are trained and queries classified) — https://qiime2.org
- **q2-feature-classifier** (Provides fit-classifier-naive-bayes (MultinomialNB with class_weight='balanced') and classify-sklearn at confidence 0.7 for the two weighting schemes) — https://github.com/qiime2/q2-feature-classifier/
- **q2-clawback** (Generates bespoke per-habitat class weights via fetch_Qiita_samples and generate_class_weights as the treatment condition) — https://github.com/BenKaehler/q2-clawback
- **scikit-learn** (Supplies the MultinomialNB classifier underlying the error-rate comparison)
- **redbiom** (Fetches environment-matched Qiita samples/features for the 14 EMPO 3 habitats used as query data) — https://github.com/biocore/redbiom
- **readytowear** (Source of precomputed habitat-specific class weights, avoiding a rerun of clawback) — https://github.com/BenKaehler/readytowear

## Examples

```
qiime feature-classifier classify-sklearn --i-classifier bespoke_classifier.qza --i-reads habitat_reads.qza --p-confidence 0.7 --o-classification predictions.qza
```

## Evaluation signals

- Overall average species-level error rate over the 14 EMPO 3 habitats falls from ~25% (uniform) to ~14% (bespoke), with paired t-test P = 5.8e-5
- Proteobacteria species-level error drops from 35.4% (±0.7% s.e.) to 22.3% (±0.4% s.e.), paired t-test P ≈ 1.4e-6
- Firmicutes species-level error drops from 43.6% (±0.7% s.e.) to 24.3% (±0.3% s.e.), paired t-test P ≈ 8.4e-6
- fig1_error_rates.csv contains paired per-habitat error rates for all 14 EMPO 3 habitats under both weighting schemes
- Both classifiers were trained on the identical Greengenes 99% reference with identical classify-sklearn confidence (0.7), isolating class weighting as the only variable

## Limitations

- Some degradation in performance was observed for rare species under bespoke weighting
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation)
- Empirical species distributions introduce potential error sources such as misannotation of sample types and sequencing biases propagated into taxonomic weights
- Cross-habitat weights still outperform uniform weights in the majority of cases, so habitat mismatch alone does not reproduce the full benefit of matched bespoke weights

## Evidence

- [abstract] overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%): "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [discussion] bespoke classification is not immune to errors that result from poorly curated reference data: "bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation)"
- [readme] ready-made weights for common primer, database, and sample habitat combinations: "If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback, please check out our ready-made weights at"
- [discussion] We demonstrated species-level classification improvement with as few as 122 samples (four-fifths of the saline surface samples from the EMP study; Table 1): "We demonstrated species-level classification improvement with as few as 122 samples (four-fifths of the saline surface samples from the EMP study; Table 1)"
