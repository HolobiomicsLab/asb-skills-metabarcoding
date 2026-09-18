---
name: naive-bayes-class-weight-estimation
description: Use when naive Bayes species-level classification in q2-feature-classifier
  needs improved accuracy for samples from an environment (e.g., EMPO 3 habitat) for
  which classified read counts from curated samples or Qiita are available.
when_to_use_negative:
- 'When no adequately characterized samples exist for the target environment: the
  article notes natural sample types lacking sufficient characterization cannot have
  bespoke weights assembled, and average or uniform weights may be preferable'
- When a ready-made weight set already exists for the primer/database/habitat combination
  (e.g., readytowear weights for animal-distal-gut, soil-non-saline, human-stool,
  etc.), running clawback is unnecessary
- When input is an unclassified feature table with no taxonomic calls — weights must
  be aggregated from classified read counts per reference taxon
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3308
tools:
- name: q2-clawback
  role: QIIME 2 plugin whose generate_class_weights action aggregates, normalizes,
    and offsets classified read counts into bespoke taxonomic weights
  repo: github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Framework hosting the plugin and the FeatureWeights artifact semantic type
    for the output class_weights.qza
- name: q2-feature-classifier
  role: Consumes the generated class weights in the naive Bayes classifier to improve
    species-level taxonomic classification
- name: redbiom
  role: Supports sample discovery and retrieval from Qiita by metadata category, feeding
    fetch_Qiita_samples for weight assembly
  repo: https://github.com/biocore/redbiom
- name: readytowear
  role: Provides pre-computed q2-clawback taxonomic weights for common primer, database,
    and EMPO 3 habitat combinations as an alternative to running clawback
  repo: https://github.com/BenKaehler/readytowear
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:9356fef80fed34ed3d9e25275834b24573fbd332c2ee44bf54ed4e2003d51475
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - generate_class_weights
  - q2-clawback
  - q2-feature-classifier
  - readytowear
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

# naive-bayes-class-weight-estimation

## Summary

Generates habitat-specific (bespoke) taxonomic class weights for QIIME 2 naive Bayes taxonomic classifiers using q2-clawback's generate_class_weights, replacing the untenable uniform-weights assumption with empirical, environment-specific species abundance distributions.

## When to use

Use when naive Bayes species-level classification in q2-feature-classifier needs improved accuracy for samples from an environment (e.g., EMPO 3 habitat) for which classified read counts from curated samples or Qiita are available.

## When NOT to use

- When no adequately characterized samples exist for the target environment: the article notes natural sample types lacking sufficient characterization cannot have bespoke weights assembled, and average or uniform weights may be preferable
- When a ready-made weight set already exists for the primer/database/habitat combination (e.g., readytowear weights for animal-distal-gut, soil-non-saline, human-stool, etc.), running clawback is unnecessary
- When input is an unclassified feature table with no taxonomic calls — weights must be aggregated from classified read counts per reference taxon

## Inputs

- Per-sample classified sequence-variant counts (FeatureTable produced by sequence_variants_from_samples on fetch_Qiita_samples output)
- Habitat metadata category (e.g., EMPO 3 habitat type per sample)
- Reference taxonomy/classification of sequences (used to attribute reads to reference taxa)

## Outputs

- class_weights.qza — a QIIME 2 artifact of semantic type FeatureWeights containing normalized, offset per-taxon weights for each habitat

## How to apply

Load per-sample classified sequence-variant counts (from sequence_variants_from_samples applied to fetch_Qiita_samples output) together with the habitat metadata category (e.g., EMPO 3). For each habitat, call q2-clawback's generate_class_weights to sum classified read counts per reference taxon across all samples in that habitat, then normalize the summed per-taxon counts to relative frequencies. Add a uniform offset (default unobserved weight 1e-6) to taxa never observed in the habitat so every reference species retains nonzero weight. Serialize each habitat's weight vector as a QIIME 2 artifact of semantic type FeatureWeights (class_weights.qza), which can then be passed to q2-feature-classifier's fit-classifier-naive-bayes with --class-weight. The rationale is that classification accuracy degrades linearly with the degree to which the equal-likelihood assumption is violated, and the more specific the weights are to the query sample's environment, the better the accuracy.

## Related tools

- **q2-clawback** (QIIME 2 plugin whose generate_class_weights action aggregates, normalizes, and offsets classified read counts into bespoke taxonomic weights) — github.com/BenKaehler/q2-clawback
- **QIIME 2** (Framework hosting the plugin and the FeatureWeights artifact semantic type for the output class_weights.qza)
- **q2-feature-classifier** (Consumes the generated class weights in the naive Bayes classifier to improve species-level taxonomic classification)
- **redbiom** (Supports sample discovery and retrieval from Qiita by metadata category, feeding fetch_Qiita_samples for weight assembly) — https://github.com/biocore/redbiom
- **readytowear** (Provides pre-computed q2-clawback taxonomic weights for common primer, database, and EMPO 3 habitat combinations as an alternative to running clawback) — https://github.com/BenKaehler/readytowear

## Examples

```
qiime clawback generate-class-weights --i-sequence-variants classified_counts.qza --m-metadata-file qiita_samples_metadata.tsv --m-metadata-column empo_3 --o-class-weights class_weights.qza
```

## Evaluation signals

- The output class_weights.qza exists and loads as a QIIME 2 artifact of semantic type FeatureWeights
- All weights are non-negative; unobserved taxa sit at the 1e-6 unobserved-weight offset while observed taxa dominate
- Per-habitat weight vectors sum (approximately) to 1 after normalization to relative frequencies
- Downstream species-level classification error with the bespoke weights is lower than with uniform weights (article: average error declines from 25% to 14%, comparable to existing classifiers' genus-level 16%)
- Each habitat's weights are derived only from samples belonging to that habitat's metadata category (e.g., each of the 14 EMPO 3 habitat types)

## Limitations

- Performance degrades for rare species under bespoke weighting
- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation) and misannotated sample types or sequencing biases propagate into the weights
- Uniform weight assumption is still negative even versus deliberately incorrect cross-habitat weights, but specificity matters: weights must match the query sample's environment
- For highly unusual sample distributions (e.g., synthetic populations), custom bespoke weights must be compiled from existing samples rather than fetched defaults

## Evidence

- [methods] q2-clawback assembles bespoke taxonomic weights: [evidence span withheld — anchor status: not_found]
- [abstract] By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types.: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [readme] If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback: "If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback, please check out our ready-made weights at [readytowear]"
- [discussion] We did observe some degradation in performance for rare species: "We did observe some degradation in performance for rare species"
