---
name: taxonomic-class-weight-assembly
description: Use when query amplicon sequences come from a sample type whose reference
  taxonomic distribution is known or estimable (e.g. human-stool, soil-non-saline,
  EMPO 3 habitats), and a naive Bayes classifier trained with uniform class weights
  is producing inflated species-level error rates.
when_to_use_negative:
- The sample type lacks sufficient characterization for bespoke weight assembly (e.g.
  a rare natural environment) — the article states that for such types, average weights
  estimated from global microbial species distributions are superior to uniform weights,
  so use precomputed 'average' weights instead
- The input is already a trained classifier or the query distribution matches the
  reference distribution well enough that uniform weights are adequate — the linear
  degradation penalty may be negligible
- You only need genus-level classification — the benefit is demonstrated mainly at
  the species level (bespoke weights give genus-level accuracy of ~14% error, comparable
  to existing classifiers' 16% genus-level error)
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: q2-clawback
  role: 'Assembles bespoke taxonomic class weights: fetch_Qiita_samples discovers
    environment-matched Qiita samples via redbiom; generate_class_weights and assemble_weights_from_Qiita
    produce the weights'
  repo: github.com/BenKaehler/q2-clawback
- name: redbiom
  role: Sample discovery backend for fetch_Qiita_samples — searches Qiita cache for
    samples by metadata queries or features within a processing context
  repo: https://github.com/biocore/redbiom
- name: q2-feature-classifier
  role: Trains the multinomial naive Bayes classifier (fit-classifier-naive-bayes
    with class_weight='balanced') under bespoke vs uniform weights and classifies
    query sequences with classify-sklearn at confidence 0.7
- name: scikit-learn
  role: Provides the MultinomialNB estimator underlying fit-classifier-naive-bayes
- name: QIIME 2
  role: Plugin framework in which q2-clawback and q2-feature-classifier actions run
    and class-weight artifacts are stored
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:ea3d2151be9f990674a26f557d64ff544642e311caefb2d2d027b0016b089abc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - imbalanced-learn
  - q2-clawback
  - q2-feature-classifier
  - redbiom
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: thin
yes_rate: 0.0
n_claims: 0
---

# taxonomic-class-weight-assembly

## Summary

Assembles environment-specific (bespoke) taxonomic class weights from empirically observed species distributions — via q2-clawback's fetch_Qiita_samples/generate_class_weights backed by redbiom, or precomputed weights from the readytowear repository — to replace the uniform-weight assumption in q2-feature-classifier naive Bayes taxonomy classifiers and cut species-level error rates (e.g. 25% → 14% across 14 EMPO 3 habitats).

## When to use

Use when query amplicon sequences come from a sample type whose reference taxonomic distribution is known or estimable (e.g. human-stool, soil-non-saline, EMPO 3 habitats), and a naive Bayes classifier trained with uniform class weights is producing inflated species-level error rates.

## When NOT to use

- The sample type lacks sufficient characterization for bespoke weight assembly (e.g. a rare natural environment) — the article states that for such types, average weights estimated from global microbial species distributions are superior to uniform weights, so use precomputed 'average' weights instead
- The input is already a trained classifier or the query distribution matches the reference distribution well enough that uniform weights are adequate — the linear degradation penalty may be negligible
- You only need genus-level classification — the benefit is demonstrated mainly at the species level (bespoke weights give genus-level accuracy of ~14% error, comparable to existing classifiers' 16% genus-level error)

## Inputs

- Environment-specific sample data / taxonomic abundance tables retrieved from Qiita via redbiom (BIOM feature tables plus sample metadata)
- Greengenes 99% OTU reference sequences and taxonomy (QIIME 2 feature-classifier reference data)
- A QIIME 2 metadata category (e.g. EMPO 3 habitat) defining sample groupings for weight assembly
- Optionally: precomputed class-weight artifacts downloaded from the readytowear repository

## Outputs

- Habitat-specific taxonomic class weights (QIIME 2 feature-classifier class-weight artifact) per environment, e.g. for the 14 EMPO 3 habitats
- Bespoke-weight and uniform-weight naive Bayes classifiers trained on the Greengenes 99% reference
- Species-level classification error rates per habitat and phylum, e.g. fig1_error_rates.csv and supplementary_fig5_error_rates.csv

## How to apply

First decide whether to fetch ready-made weights from the readytowear repository (searchable via its inventory.tsv) or run clawback yourself. To assemble weights, use q2-clawback's fetch_Qiita_samples to discover environment-matched samples via redbiom (e.g. against the Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4 context), optionally summarize_Qiita_metadata_category_and_contexts and sequence_variants_from_samples, then generate_class_weights to compute per-taxon class weights; assemble_weights_from_Qiita chains these steps. Train the multinomial naive Bayes classifier once with fit-classifier-naive-bayes using class_weight='balanced' bespoke weights and once with uniform weights, then classify held-out sequence variants with classify-sklearn at confidence 0.7 and compare species-level error rates. The rationale is that classification accuracy degrades linearly as the equal-abundance assumption is violated, and even deliberately incorrect (cross-habitat) weights outperform uniform weights for species classification in the majority of cases.

## Related tools

- **q2-clawback** (Assembles bespoke taxonomic class weights: fetch_Qiita_samples discovers environment-matched Qiita samples via redbiom; generate_class_weights and assemble_weights_from_Qiita produce the weights) — github.com/BenKaehler/q2-clawback
- **redbiom** (Sample discovery backend for fetch_Qiita_samples — searches Qiita cache for samples by metadata queries or features within a processing context) — https://github.com/biocore/redbiom
- **q2-feature-classifier** (Trains the multinomial naive Bayes classifier (fit-classifier-naive-bayes with class_weight='balanced') under bespoke vs uniform weights and classifies query sequences with classify-sklearn at confidence 0.7)
- **scikit-learn** (Provides the MultinomialNB estimator underlying fit-classifier-naive-bayes)
- **QIIME 2** (Plugin framework in which q2-clawback and q2-feature-classifier actions run and class-weight artifacts are stored)

## Examples

```
redbiom search metadata 'EMPO_3=human-stool' | redbiom fetch samples --context Pick_closed-reference_OTUs-Greengenes-Illumina-16S-V4-5c6506 --output stool_samples.biom
```

## Evaluation signals

- Bespoke weighting should drop overall species-level error rates from ~25% (uniform) to ~14% across the 14 EMPO 3 habitats, with paired t-test P = 5.8e-5
- Per-phylum validation: Proteobacteria error falls from ~35.4% (±0.7% s.e.) to ~22.3% (±0.4% s.e.) (P ≈ 1.4e-6) and Firmicutes from ~43.6% to ~24.3% (P ≈ 8.4e-6)
- The assembled weights must cover the taxa in the Greengenes 99% reference taxonomy so that fit-classifier-naive-bayes accepts them as class_weight input
- A sanity-check improvement should appear even with deliberately incorrect (cross-habitat) weights versus uniform weights for species classification in the majority of cases
- If using readytowear instead of running clawback, confirm the weight set matches the primer (e.g. 515f-806r trimmed vs full-length), reference database, and EMPO 3 habitat of the query samples via the repository's inventory.tsv

## Limitations

- Bespoke classification is not immune to errors from poorly curated reference data (e.g., reference sequence misannotation)
- Empirical species distributions introduce error sources such as misannotation of sample types and sequencing biases propagated into the taxonomic weights; systematic sequencing-technology bias is not controlled without integrating multiple technologies
- Some degradation in performance was observed for rare species under bespoke weighting
- redbiom's default host (qiita.ucsd.edu:7329) is read-only; loading custom data requires a local Redis/Webdis instance and an explicit REDBIOM_HOST setting

## Evidence

- [abstract] By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types.: "By incorporating environment-specific taxonomic abundance information, we demonstrate a significant increase in the species-level classification accuracy across common sample types."
- [methods] readytowear shortcut: "If you like the idea of using taxonomic weights to improve your taxonomic classification accuracy, but don't want to run clawback"
- [discussion] Qiita weight assembly feasibility: "it is trivial to assemble weights for all stool samples with human hosts from Qiita"
- [readme] readytowear provenance: [evidence span withheld — anchor status: external_doc]
- [discussion] rare species caveat: "We did observe some degradation in performance for rare species"
