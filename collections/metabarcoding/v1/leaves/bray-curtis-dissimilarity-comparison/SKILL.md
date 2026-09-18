---
name: bray-curtis-dissimilarity-comparison
description: Use when you have predicted taxonomies and known ground-truth taxonomies
  for matched test samples and need to assess whether a classification condition (e.g.,
  species-level bespoke weights vs. genus-level uniform weights) shifts community
  composition, not just per-read error.
when_to_use_negative:
- Predicted and ground-truth taxonomies have not been truncated to a shared rank/label
  space (species vs. genus) — Bray-Curtis over mismatched label spaces is meaningless
- Only per-sequence classification error rate is of interest; Bray-Curtis measures
  composition shifts and is the complement to (not substitute for) the error-rate
  t-test
- Samples are not matched across conditions — the single-sided paired t-test design
  requires matched predicted-vs-true pairs per sample
edam_operation: http://edamontology.org/operation_0304
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3697
tools:
- name: q2-feature-classifier
  role: classify-sklearn produces the predicted per-sample taxonomies under each weighting
    condition that are converted to compositions for Bray-Curtis comparison
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: QIIME 2
  role: provides the Bray-Curtis beta-diversity metric used to compute per-sample
    dissimilarity between predicted and true communities
- name: scipy
  role: alternative implementation of the Bray-Curtis dissimilarity metric for per-sample
    composition comparison
- name: q2-clawback
  role: assembles the bespoke taxonomic weights that define the bespoke-species classification
    condition being benchmarked
  repo: https://github.com/BenKaehler/q2-clawback
- name: readytowear
  role: supplies ready-made EMPO 3 habitat weights (e.g., animal-distal-gut, soil-non-saline)
    and an 'average' classifier over 14 EMPO habitats for use with the classifier
  repo: https://github.com/BenKaehler/readytowear
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:933a396ad6d0e31c63c7358eae3390b86b150128ecd00c02102fddce98009e41
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
  - scipy
  edam_operation: http://edamontology.org/operation_0304
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# bray-curtis-dissimilarity-comparison

## Summary

A benchmarking pattern from Kaehler et al. (2019, Nat Commun 10:4643) that quantifies how faithfully a taxonomic classifier reproduces true community composition by computing Bray-Curtis dissimilarity between predicted and ground-truth relative abundances, then comparing classification conditions with a single-sided paired t-test.

## When to use

Use when you have predicted taxonomies and known ground-truth taxonomies for matched test samples and need to assess whether a classification condition (e.g., species-level bespoke weights vs. genus-level uniform weights) shifts community composition, not just per-read error.

## When NOT to use

- Predicted and ground-truth taxonomies have not been truncated to a shared rank/label space (species vs. genus) — Bray-Curtis over mismatched label spaces is meaningless
- Only per-sequence classification error rate is of interest; Bray-Curtis measures composition shifts and is the complement to (not substitute for) the error-rate t-test
- Samples are not matched across conditions — the single-sided paired t-test design requires matched predicted-vs-true pairs per sample

## Inputs

- Predicted per-sample taxonomic profiles from classify-sklearn (q2-feature-classifier, Greengenes 13_8 99% OTU reference)
- Ground-truth (known) taxonomic compositions for matched test samples from the 14 EMPO 3 habitat types
- Pre-fitted naive Bayes classifier plus bespoke class weights (readytowear) and uniform weights
- Test amplicon sequence collections per sample

## Outputs

- Per-sample Bray-Curtis dissimilarity values for each classification condition
- Mean Bray-Curtis dissimilarity per condition (target: ~0.13 bespoke species vs. ~0.15 uniform genus)
- Single-sided paired t-statistic and P-value across matched samples (target P = 0.013)
- fig1_comparison.csv compiling error rates, dissimilarities, t-statistics, and P-values

## How to apply

Classify the same test amplicon sequences under each condition with q2-feature-classifier's classify-sklearn against the Greengenes 13_8 99% OTU reference — species-level classification with bespoke Qiita-derived weights, genus-level with uniform weights. For each matched sample, convert predicted and true taxonomic assignments to relative abundances on a shared, consistently truncated label space (species for bespoke, genus for uniform) so Bray-Curtis is computed on comparable compositions. Compute Bray-Curtis dissimilarity per sample per condition using the Bray-Curtis metric in QIIME 2 (beta diversity) or scipy.spatial.distance.braycurtis. Then run a single-sided paired t-test comparing dissimilarity distributions across matched samples between conditions, and compile mean dissimilarities, t-statistics, and P-values (the paper's Fig. 1 benchmark: 0.13 bespoke-species vs. 0.15 uniform-genus, P = 0.013) into fig1_comparison.csv.

## Related tools

- **q2-feature-classifier** (classify-sklearn produces the predicted per-sample taxonomies under each weighting condition that are converted to compositions for Bray-Curtis comparison) — https://github.com/qiime2/q2-feature-classifier/
- **QIIME 2** (provides the Bray-Curtis beta-diversity metric used to compute per-sample dissimilarity between predicted and true communities)
- **scipy** (alternative implementation of the Bray-Curtis dissimilarity metric for per-sample composition comparison)
- **q2-clawback** (assembles the bespoke taxonomic weights that define the bespoke-species classification condition being benchmarked) — https://github.com/BenKaehler/q2-clawback
- **readytowear** (supplies ready-made EMPO 3 habitat weights (e.g., animal-distal-gut, soil-non-saline) and an 'average' classifier over 14 EMPO habitats for use with the classifier) — https://github.com/BenKaehler/readytowear

## Examples

```
qiime diversity beta --i-table table.qza --p-metric braycurtis --o-distance-matrix braycurtis_dm.qza
```

## Evaluation signals

- Mean Bray-Curtis dissimilarity ≈ 0.13 for species-level bespoke weights and ≈ 0.15 for genus-level uniform weights, with single-sided paired t-test P = 0.013 (paper Fig. 1)
- Companion error rates reproduce ~14% (bespoke species) vs. ~16% (uniform genus) across the 14 EMPO 3 habitats
- Every dissimilarity value lies in [0, 1] and there is exactly one value per matched test sample per condition
- Predicted and true taxonomies are truncated consistently per condition (species for bespoke, genus for uniform) before metric computation
- t-statistics and P-values are compiled in fig1_comparison.csv alongside per-condition means

## Limitations

- Bespoke weighting showed degradation in performance for rare species
- Bespoke classification is not immune to errors from poorly curated reference data, e.g., reference sequence misannotation
- Empirical species distributions can propagate misannotated sample types and sequencing biases into the weights being benchmarked

## Evidence

- [abstract] Species-level error benchmark: "At the species level, overall average error rates decline from 25% to 14%, which is favourably comparable to the error rates that existing classifiers achieve at the genus level (16%)."
- [discussion] Sample-size grounding for habitat-specific benchmarking: "We demonstrated species-level classification improvement with as few as 122 samples (four-fifths of the saline surface samples from the EMP study; Table 1)"
- [readme] Ready-made weights used with the classifier: [evidence span withheld — anchor status: external_doc]
- [readme] Average classifier over 14 EMPO habitats: [evidence span withheld — anchor status: external_doc]
- [discussion] Reference-data caveat for bespoke condition: "bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation)"
