---
name: amplicon-read-count-aggregation
description: Use when you have per-sample classified sequence-variant read counts
  over a reference taxonomy plus a habitat metadata category (e.g., EMPO 3), and you
  need habitat-specific taxonomic weights for q2-feature-classifier.
when_to_use_negative:
- The sample habitat lacks sufficient characterization for bespoke weight assembly
  — for such natural sample types, average weights estimated from global microbial
  species distributions are superior to uniform weights.
- The input is a highly unusual sample distribution (e.g., synthetic populations)
  with no existing curated samples — compile custom bespoke weights from existing
  samples instead of aggregating generic counts.
- You already have ready-made weights matching your primer, database, and habitat
  combination (readytowear covers common combinations) — aggregating your own counts
  is unnecessary.
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: q2-clawback
  role: Provides generate_class_weights, which sums, normalizes, and offsets classified
    read counts into habitat-specific FeatureWeights artifacts
  repo: https://github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Plugin framework in which q2-clawback runs; defines the FeatureWeights artifact
    serialization
  repo: https://qiime2.org
- name: q2-feature-classifier
  role: 'Downstream consumer of the aggregated weights: the FeatureWeights artifact
    is used to fit/weight its naive Bayes taxonomy classifier'
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: readytowear
  role: Repository of pre-aggregated ready-made weights produced by this skill for
    common primer/database/habitat combinations
  repo: https://github.com/BenKaehler/readytowear
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:4ac9c9652d02b4e174616eab59cbb8f2e479f36a7c245b750628efe32896b33f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - q2-clawback
  - q2-feature-classifier
  - readytowear
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# amplicon-read-count-aggregation

## Summary

Aggregates per-sample classified amplicon read counts into habitat-specific (bespoke) taxonomic weight vectors using q2-clawback's generate_class_weights, producing FeatureWeights QIIME 2 artifacts that raise species-level classification accuracy of q2-feature-classifier over the uniform-weight assumption.

## When to use

Use when you have per-sample classified sequence-variant read counts over a reference taxonomy plus a habitat metadata category (e.g., EMPO 3), and you need habitat-specific taxonomic weights for q2-feature-classifier.

## When NOT to use

- The sample habitat lacks sufficient characterization for bespoke weight assembly — for such natural sample types, average weights estimated from global microbial species distributions are superior to uniform weights.
- The input is a highly unusual sample distribution (e.g., synthetic populations) with no existing curated samples — compile custom bespoke weights from existing samples instead of aggregating generic counts.
- You already have ready-made weights matching your primer, database, and habitat combination (readytowear covers common combinations) — aggregating your own counts is unnecessary.

## Inputs

- per-sample classified sequence-variant counts (taxon x sample table from sequence_variants_from_samples)
- sample metadata file with a habitat category column (e.g., EMPO 3 habitat type)
- reference taxonomy / classification of sequence variants against the reference database (e.g., Greengenes or RefSeq)

## Outputs

- class_weights.qza — QIIME 2 artifact of semantic type FeatureWeights, one habitat-specific weight vector per habitat
- per-habitat normalized, offset taxonomic weight vectors (relative frequencies; unobserved taxa at 1e-6)

## How to apply

Load the per-sample classified sequence-variant counts (from sequence_variants_from_samples applied to fetch_Qiita_samples output) together with the habitat metadata category, e.g., the 14 EMPO 3 habitat types. For each habitat, call q2-clawback's generate_class_weights to sum classified read counts per reference taxon across all samples in that habitat, then normalize the summed per-taxon counts to relative frequencies. Add a uniform offset — the default unobserved weight of 1e-6 — to taxa never observed in the habitat so every reference species retains nonzero weight. Serialize each habitat's weight vector as a QIIME 2 artifact of semantic type FeatureWeights, e.g., class_weights.qza, ready to be passed to classifier fitting with q2-feature-classifier. The rationale is that naive Bayes classifiers assume all reference species are equally likely to be observed, and accuracy degrades linearly as that assumption is violated.

## Related tools

- **q2-clawback** (Provides generate_class_weights, which sums, normalizes, and offsets classified read counts into habitat-specific FeatureWeights artifacts) — https://github.com/BenKaehler/q2-clawback
- **QIIME 2** (Plugin framework in which q2-clawback runs; defines the FeatureWeights artifact serialization) — https://qiime2.org
- **q2-feature-classifier** (Downstream consumer of the aggregated weights: the FeatureWeights artifact is used to fit/weight its naive Bayes taxonomy classifier) — https://github.com/qiime2/q2-feature-classifier/
- **readytowear** (Repository of pre-aggregated ready-made weights produced by this skill for common primer/database/habitat combinations) — https://github.com/BenKaehler/readytowear

## Examples

```
qiime clawback generate-class-weights --i-classification classification.qza --m-metadata-file empo3.tsv --m-metadata-column empo_3 --p-unobserved-weight 1e-6 --o-classification-weights class_weights.qza
```

## Evaluation signals

- The output artifact exists and loads as a QIIME 2 artifact of semantic type FeatureWeights.
- All contained weights are non-negative, with observed taxa dominating and taxa never observed in the habitat sitting at the 1e-6 offset level.
- Every taxon in the reference has nonzero weight (the uniform offset guarantees no zero-weight species).
- Per-habitat weight vectors sum to normalized relative frequencies (each sums to ~1).
- Species-level classification accuracy with these weights should improve over uniform weights (reported average error rate declines from 25% to 14%).

## Limitations

- Some degradation in performance was observed for rare species under bespoke weighting.
- Empirical species distributions introduce potential error sources such as misannotated sample types and sequencing biases propagated into the taxonomic weights.
- Natural sample types lacking sufficient characterization cannot have bespoke weights assembled; systematic sequencing-technique bias is not controlled without integrating multiple technologies for microbial distribution estimation.
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation).

## Evidence

- [abstract] environment-specific weights improve species-level accuracy: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [discussion] weight specificity drives accuracy: "demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy"
- [discussion] rare species degrade under bespoke weighting: "We did observe some degradation in performance for rare species"
- [readme] readytowear weights are the pre-computed output of this skill: [evidence span withheld — anchor status: external_doc]
- [readme] weights improve classification but require running clawback: "If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback, please check out our ready-made weights at [readytowear]"
