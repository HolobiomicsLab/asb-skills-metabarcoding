---
name: kmer-bray-curtis-dissimilarity-sampling
description: Use when you need to explain per-habitat species-level error rates of
  bespoke (q2-clawback) taxonomic weights by how often similar marker-gene sequences
  in the same habitat originate from different species, e.g. reproducing the paper's
  Fig. 5 confusion-index analysis.
when_to_use_negative:
- No bespoke taxonomic weights exist for the habitat — natural sample types lacking
  sufficient characterization cannot have bespoke weights assembled
- The goal is taxonomic classification of new sequences itself, rather than characterizing
  why classification errors occur in a habitat
- You need absolute accuracy metrics rather than an explanatory statistic correlating
  error rate with within-habitat sequence confusion
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3697
tools:
- name: q2-clawback
  role: generate_class_weights produces the per-habitat bespoke taxonomic weights
    whose weighted reference distributions are sampled for the confusion-index computation
  repo: github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: framework hosting the q2-clawback and q2-feature-classifier plugins used to
    produce weights and species-level error rates
  repo: https://qiime2.org
- name: q2-feature-classifier
  role: the naive Bayes classifier whose bespoke species-level classification error
    rates are joined with the confusion indices
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: scikit-learn
  role: used to fit the linear regression of per-habitat error rate on confusion index
    and report Pearson r2 and P-value
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:7e510f3d0e8758bf5530a59f495ded165e4b0603162199c67ddb5147afcdba3c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - q2-clawback
  - q2-feature-classifier
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# kmer-bray-curtis-dissimilarity-sampling

## Summary

Quantifies per-habitat taxonomic confusability by sampling 10^8 pairs of sequences from each habitat's bespoke-weighted reference distribution, retaining pairs with k-mer Bray-Curtis dissimilarity < 0.25, and computing the confusion index (log of expected Greengenes taxonomic distance), which is correlated with per-habitat species-level classification error rates (Pearson r2 = 0.72, P = 1.4e-4).

## When to use

Use when you need to explain per-habitat species-level error rates of bespoke (q2-clawback) taxonomic weights by how often similar marker-gene sequences in the same habitat originate from different species, e.g. reproducing the paper's Fig. 5 confusion-index analysis.

## When NOT to use

- No bespoke taxonomic weights exist for the habitat — natural sample types lacking sufficient characterization cannot have bespoke weights assembled
- The goal is taxonomic classification of new sequences itself, rather than characterizing why classification errors occur in a habitat
- You need absolute accuracy metrics rather than an explanatory statistic correlating error rate with within-habitat sequence confusion

## Inputs

- Per-habitat bespoke taxonomic weights generated with q2-clawback's generate_class_weights
- Greengenes reference taxonomy (used to measure taxonomic distance between species)
- Per-habitat species-level classification error rates measured on held-out test samples
- Sequence data / weighted reference distributions per habitat

## Outputs

- Per-habitat confusion indices (log of expected taxonomic distance over retained pairs)
- Pearson r2 and P-value from the linear regression of error rate on confusion index
- confusion_index_vs_error.csv containing confusion indices, error rates, and correlation statistics

## How to apply

First load the per-habitat bespoke taxonomic weights generated with q2-clawback's generate_class_weights (or the equivalent readytowear weights) for the environments being evaluated. For each habitat, sample 10^8 pairs of sequences from that habitat's weighted reference distribution and retain only pairs with k-mer Bray-Curtis dissimilarity < 0.25, so that only near-identical sequences enter the analysis. For each habitat compute the confusion index as the log of the expected taxonomic distance over the retained sequence pairs, measuring distance between taxonomies with the Greengenes taxonomy. Join the per-habitat confusion indices with the per-habitat bespoke species-level classification error rates measured on held-out test samples, fit a linear regression of error rate on confusion index, and report Pearson r2 and P-value. Write the confusion indices, error rates, and correlation statistics to confusion_index_vs_error.csv, checking agreement with the reported r2 = 0.72 (P = 1.4e-4).

## Related tools

- **q2-clawback** (generate_class_weights produces the per-habitat bespoke taxonomic weights whose weighted reference distributions are sampled for the confusion-index computation) — github.com/BenKaehler/q2-clawback
- **QIIME 2** (framework hosting the q2-clawback and q2-feature-classifier plugins used to produce weights and species-level error rates) — https://qiime2.org
- **q2-feature-classifier** (the naive Bayes classifier whose bespoke species-level classification error rates are joined with the confusion indices) — https://github.com/qiime2/q2-feature-classifier/
- **scikit-learn** (used to fit the linear regression of per-habitat error rate on confusion index and report Pearson r2 and P-value)

## Examples

```
python -c "import pandas as pd; from scipy.stats import pearsonr; d=pd.read_csv('confusion_index_vs_error.csv'); print(pearsonr(d.confusion_index, d.error_rate))"
```

## Evaluation signals

- Reproduced Pearson correlation between confusion index and per-habitat bespoke species-level error rate approximates r2 = 0.72 (P = 1.4e-4), as reported for Fig. 5
- Only sequence pairs with k-mer Bray-Curtis dissimilarity < 0.25 are retained before computing the confusion index
- Exactly 10^8 pairs sampled per habitat from the weighted reference distribution
- Confusion index values equal the log of the expected taxonomic distance computed against the Greengenes taxonomy
- confusion_index_vs_error.csv exists and contains one row per habitat with confusion index, error rate, and correlation statistics

## Limitations

- Bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation), which the empirical species distributions may propagate into the weights
- Misannotation of sample types or sequencing biases are potential error sources when using empirical species distributions
- The correlation leaves ~28% of error-rate variance unexplained (r2 = 0.72), so the confusion index explains but does not fully determine bespoke classification accuracy

## Evidence

- [methods] pair sampling threshold: [evidence span withheld — anchor status: not_found]
- [methods] confusion index definition: [evidence span withheld — anchor status: not_found]
- [results] Fig. 5 correlation finding: [evidence span withheld — anchor status: not_found]
- [discussion] reference-data caveat: "bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation)"
- [readme] weights provenance: [evidence span withheld — anchor status: external_doc]
