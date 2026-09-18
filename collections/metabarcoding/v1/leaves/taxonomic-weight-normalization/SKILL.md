---
name: taxonomic-weight-normalization
description: Use when classified sequence-variant counts from environment-specific
  samples (e.g., EMPO 3 habitats fetched from Qiita) are available and habitat-specific
  FeatureWeights artifacts are needed to improve species-level naive Bayes classification
  accuracy.
when_to_use_negative:
- The sample's source environment lacks sufficient characterization for bespoke weight
  assembly — in that case average weights estimated from global microbial species
  distributions are superior to uniform weights, and ready-made weights (readytowear)
  may be preferable
- The sample has a highly unusual distribution (e.g., synthetic populations) where
  existing habitat weights would be deliberately incorrect — compile custom bespoke
  weights from existing samples instead
- You only need generic, non-habitat-specific classification and cannot justify the
  empirical distribution's error sources (misannotated sample types, sequencing biases
  propagated into weights)
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: q2-clawback
  role: Provides generate_class_weights (and assemble_weights_from_Qiita) to aggregate,
    normalize, and offset classified read counts into habitat-specific FeatureWeights
    artifacts
  repo: github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Plugin framework hosting q2-clawback and providing the FeatureWeights artifact
    semantic type and q2-feature-classifier integration
- name: redbiom
  role: Sample discovery/retrieval backend used via fetch_Qiita_samples to obtain
    per-sample feature data for weight assembly
  repo: biocore/redbiom
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:af85abfa20440787768f9ef40f70afebc09ec4aafc6e8fab3a6be60209642533
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - q2-clawback
  - redbiom
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# taxonomic-weight-normalization

## Summary

Generates habitat-specific (bespoke) taxonomic weights for q2-feature-classifier by aggregating per-sample classified read counts with q2-clawback's generate_class_weights, normalizing them to relative frequencies, and offsetting unobserved taxa so every reference species retains nonzero weight.

## When to use

Use when classified sequence-variant counts from environment-specific samples (e.g., EMPO 3 habitats fetched from Qiita) are available and habitat-specific FeatureWeights artifacts are needed to improve species-level naive Bayes classification accuracy.

## When NOT to use

- The sample's source environment lacks sufficient characterization for bespoke weight assembly — in that case average weights estimated from global microbial species distributions are superior to uniform weights, and ready-made weights (readytowear) may be preferable
- The sample has a highly unusual distribution (e.g., synthetic populations) where existing habitat weights would be deliberately incorrect — compile custom bespoke weights from existing samples instead
- You only need generic, non-habitat-specific classification and cannot justify the empirical distribution's error sources (misannotated sample types, sequencing biases propagated into weights)

## Inputs

- Per-sample classified sequence-variant counts (FeatureTable from sequence_variants_from_samples on fetch_Qiita_samples output)
- Sample metadata category defining the environment/habitat (e.g., EMPO 3 habitat type from Qiita)

## Outputs

- class_weights.qza — a QIIME 2 artifact of semantic type FeatureWeights containing per-habitat taxonomic weight vectors with unobserved taxa at the 1e-6 offset

## How to apply

Load the per-sample classified sequence-variant counts (produced by sequence_variants_from_samples on fetch_Qiita_samples output) together with the habitat metadata category (e.g., EMPO 3). For each habitat, call q2-clawback's generate_class_weights to sum classified read counts per reference taxon across all samples in that habitat, then normalize the summed per-taxon counts to relative frequencies. Add a uniform offset (default unobserved weight 1e-6) to taxa never observed in that habitat so every reference species retains nonzero weight. Serialize each habitat's weight vector as a QIIME 2 artifact of semantic type FeatureWeights (class_weights.qza), which can then feed q2-feature-classifier's fit_classifier_naive_bayes with class weights. The rationale is that naive Bayes classifiers assume all reference species are equally likely to be observed — an assumption the article shows is 'in practice it is always violated' and whose violation degrades accuracy linearly; the more specific the weights are to the query sample's environment, the better the classification accuracy.

## Related tools

- **q2-clawback** (Provides generate_class_weights (and assemble_weights_from_Qiita) to aggregate, normalize, and offset classified read counts into habitat-specific FeatureWeights artifacts) — github.com/BenKaehler/q2-clawback
- **QIIME 2** (Plugin framework hosting q2-clawback and providing the FeatureWeights artifact semantic type and q2-feature-classifier integration)
- **redbiom** (Sample discovery/retrieval backend used via fetch_Qiita_samples to obtain per-sample feature data for weight assembly) — biocore/redbiom

## Examples

```
qiime clawback generate-class-weights --i-classification fetch_Qiita_samples_taxonomy.qza --m-metadata-file qiita_metadata.tsv --m-metadata-column empo_3 --o-class-weights class_weights.qza
```

## Evaluation signals

- class_weights.qza exists and loads as a QIIME 2 artifact of semantic type FeatureWeights
- All weights in each habitat's vector are non-negative
- Observed taxa dominate the weight distribution while unobserved taxa sit at the uniform 1e-6 offset level
- Per-habitat weight vectors sum to 1 after normalization (relative frequencies) before offsetting
- Downstream species-level classification error with the bespoke weights should be lower than with uniform weights (article reports average species-level error dropping from 25% to 14%)

## Limitations

- Some degradation in performance was observed for rare species under bespoke weighting
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation) and from misannotated sample types or sequencing biases propagated into the empirical species distributions
- Weights from a mismatched habitat are deliberately incorrect yet still often beat uniform weights — but the more specific the weights are to the query sample's environment, the better the accuracy, so habitat misassignment reduces gains
- Natural sample types lacking sufficient characterization cannot have bespoke weights assembled

## Evidence

- [abstract] By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types.: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [discussion] Uniform weight assumption harms classification performance.: "We have demonstrated that this assumption imposes a consistently negative impact on performance, even when compared to deliberately incorrect taxonomic weights"
- [discussion] More specific weights improve accuracy.: "demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy"
- [abstract] q2-clawback is an alternative for common environments.: "q2-clawback provides a straightforward alternative for samples from common environments."
- [readme] Ready-made weights from q2-clawback are available.: [evidence span withheld — anchor status: external_doc]
