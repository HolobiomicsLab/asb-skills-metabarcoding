---
name: taxonomic-distance-computation
description: Use when you need to explain per-habitat species-level classification
  error rates (e.g., from bespoke q2-clawback weights) by measuring how often similar
  sequences from the same habitat originate from different species.
when_to_use_negative:
- The question is sequence similarity (k-mer/alignment distance) rather than how often
  taxonomically distinct species produce similar sequences in a habitat
- No habitat-specific weighted reference distribution exists (natural sample types
  lacking sufficient characterization cannot have bespoke weights assembled)
- Evaluating genus-level or higher-rank classification error, where the confusion
  index's species-level distance signal is not the relevant quantity
tools:
- name: q2-clawback
  role: Generates the per-habitat bespoke taxonomic weights whose weighted reference
    distributions are sampled for sequence pairs
  repo: github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Framework hosting the q2-clawback and q2-feature-classifier plugins used to
    produce weights and error rates
- name: q2-feature-classifier
  role: Provides the naive Bayes taxonomy classifier whose bespoke per-habitat error
    rates are explained by the confusion index
- name: scikit-learn
  role: Used to fit the linear regression and compute Pearson r2 and P-value between
    confusion index and error rate
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:052529f88f0b23c71764d2da36288915554553cba3c4a7fcf112764fe50fbde8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QIIME 2
  - q2-clawback
  - q2-feature-classifier
  - scikit-learn
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# taxonomic-distance-computation

## Summary

Computes per-habitat 'confusion indices' — the log of expected taxonomic distance between closely similar marker-gene sequence pairs drawn from a habitat's weighted reference distribution — to quantify how often sequences from different species in the same habitat can be confused.

## When to use

Use when you need to explain per-habitat species-level classification error rates (e.g., from bespoke q2-clawback weights) by measuring how often similar sequences from the same habitat originate from different species.

## When NOT to use

- The question is sequence similarity (k-mer/alignment distance) rather than how often taxonomically distinct species produce similar sequences in a habitat
- No habitat-specific weighted reference distribution exists (natural sample types lacking sufficient characterization cannot have bespoke weights assembled)
- Evaluating genus-level or higher-rank classification error, where the confusion index's species-level distance signal is not the relevant quantity

## Inputs

- Per-habitat bespoke taxonomic weights generated with q2-clawback generate_class_weights
- Weighted reference sequence/taxonomy distributions per habitat (e.g., Greengenes reference with 515f-806r or full-length reads)
- Per-habitat species-level classification error rates on held-out test samples

## Outputs

- confusion_index_vs_error.csv containing per-habitat confusion indices, error rates, and correlation statistics
- Linear regression fit with Pearson r2 and P-value

## How to apply

Load the per-habitat bespoke taxonomic weights produced by q2-clawback's generate_class_weights for the environments under evaluation. For each habitat, sample 10^8 pairs of sequences from the habitat's weighted reference distribution and retain only pairs with k-mer Bray-Curtis dissimilarity < 0.25 — this restricts the analysis to sequence pairs a classifier could plausibly confuse. For each retained pair, measure the distance between the two source taxonomies using the Greengenes taxonomy, and compute the habitat's confusion index as the log of the expected taxonomic distance over retained pairs. Join these per-habitat confusion indices with the per-habitat bespoke species-level error rates measured on held-out test samples, then fit a linear regression of error rate on confusion index to obtain Pearson r2 and P-value.

## Related tools

- **q2-clawback** (Generates the per-habitat bespoke taxonomic weights whose weighted reference distributions are sampled for sequence pairs) — github.com/BenKaehler/q2-clawback
- **QIIME 2** (Framework hosting the q2-clawback and q2-feature-classifier plugins used to produce weights and error rates)
- **q2-feature-classifier** (Provides the naive Bayes taxonomy classifier whose bespoke per-habitat error rates are explained by the confusion index)
- **scikit-learn** (Used to fit the linear regression and compute Pearson r2 and P-value between confusion index and error rate)

## Examples

```
from scipy.spatial.distance import braycurtis; d = braycurtis(kmers_a, kmers_b)  # retain pair only if d < 0.25
```

## Evaluation signals

- Reproduced Pearson correlation between per-habitat confusion index and bespoke species-level error rate approximates r2 = 0.72 with P = 1.4e-4 (as reported for Fig. 5)
- Each habitat's pair sampling used 10^8 pairs, with only pairs of k-mer Bray-Curtis dissimilarity < 0.25 retained
- Confusion index equals the log of the expected taxonomic distance over retained pairs, with distances measured on the Greengenes taxonomy
- confusion_index_vs_error.csv contains one row per habitat with confusion index, error rate, and regression statistics
- Regression is fitted with error rate as the dependent variable and confusion index as the predictor

## Limitations

- Only pairs with k-mer Bray-Curtis dissimilarity < 0.25 are retained, so the index ignores distant sequence pairs
- Taxonomic distance is defined by the Greengenes taxonomy, so index values depend on that reference taxonomy's curation
- Bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation), which the confusion index does not capture
- Empirical species distributions can propagate misannotation of sample types or sequencing biases into the taxonomic weights underlying the pair sampling

## Evidence

- [results] Error rates for bespoke weights were correlated with the confusion index (Pearson r2 = 0.72, P = 1.4 × 10−4): [evidence span withheld — anchor status: not_found]
- [methods] Sample 10^8 pairs ... retain only pairs with k-mer Bray-Curtis dissimilarity < 0.25: [evidence span withheld — anchor status: not_found]
- [methods] Compute the confusion index per habitat as the log of the expected taxonomic distance: [evidence span withheld — anchor status: not_found]
- [abstract] Classification accuracy degrades linearly with the degree to which that assumption is violated: "We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated."
- [readme] Ready-made taxonomic weights generated by q2-clawback for use with the q2-feature-classifier taxonomy classifier: [evidence span withheld — anchor status: external_doc]
