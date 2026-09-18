---
name: confusion-index-estimation-from-weights
description: Use when you have per-habitat bespoke taxonomic weights (from q2-clawback
  generate_class_weights) and per-habitat species-level classification error rates,
  and need to test whether habitat-specific sequence confusability explains those
  error rates.
when_to_use_negative:
- The sample's habitat lacks sufficient characterization for bespoke weight assembly
  — for such natural sample types the article recommends average weights estimated
  from global microbial species distributions instead, so there is no weighted reference
  distribution to sample pairs from.
- You need to improve classification directly rather than explain its error structure
  — this skill is a diagnostic of error rates, not a classifier; use q2-feature-classifier
  with the habitat's bespoke weights for classification itself.
- Only a single habitat is available — the skill requires joining per-habitat confusion
  indices with per-habitat error rates across multiple environments to fit the correlation.
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0091
tools:
- name: q2-clawback
  role: Generates the per-habitat bespoke taxonomic weights whose weighted reference
    distribution is sampled to estimate the confusion index
  repo: https://github.com/BenKaehler/q2-clawback
- name: QIIME 2
  role: Plugin framework under which q2-clawback weights and q2-feature-classifier
    classification (and thus the per-habitat error rates) are produced
- name: q2-feature-classifier
  role: Produces the bespoke species-level classifications whose per-habitat error
    rates are correlated against the confusion index
  repo: https://github.com/qiime2/q2-feature-classifier/
- name: scikit-learn
  role: Used to fit the linear regression and compute the Pearson r2 and P-value between
    confusion index and error rate
- name: redbiom
  role: Fetches the habitat-specific (e.g., EMPO 3) samples from Qiita used to assemble
    the bespoke weights
  repo: biocore/redbiom
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-12669-6
    title: Species abundance information improves sequence taxonomy classification
      accuracy
  source_content_hash: sha256:010597b3b5c9df7ae294ec390dfcbfbe6b93fe4a374195840190dee0582f7f14
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BenKaehler/q2-clawback
  tools:
  - QIIME 2
  - q2-clawback
  - q2-feature-classifier
  - redbiom
  - scikit-learn
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# confusion-index-estimation-from-weights

## Summary

Estimates a per-habitat confusion index — the log of the expected taxonomic distance between Greengenes-labeled species pairs that are k-mer-similar — by sampling sequence pairs from bespoke (clawback-derived) taxonomic weights, then tests how well it predicts per-habitat species-level classification error rates.

## When to use

Use when you have per-habitat bespoke taxonomic weights (from q2-clawback generate_class_weights) and per-habitat species-level classification error rates, and need to test whether habitat-specific sequence confusability explains those error rates.

## When NOT to use

- The sample's habitat lacks sufficient characterization for bespoke weight assembly — for such natural sample types the article recommends average weights estimated from global microbial species distributions instead, so there is no weighted reference distribution to sample pairs from.
- You need to improve classification directly rather than explain its error structure — this skill is a diagnostic of error rates, not a classifier; use q2-feature-classifier with the habitat's bespoke weights for classification itself.
- Only a single habitat is available — the skill requires joining per-habitat confusion indices with per-habitat error rates across multiple environments to fit the correlation.

## Inputs

- Per-habitat bespoke taxonomic weights (q2-clawback generate_class_weights outputs for each evaluated environment)
- Greengenes reference taxonomy (used to compute taxonomic distance between species pairs)
- Per-habitat species-level classification error rates measured on held-out test samples

## Outputs

- Per-habitat confusion index values (log expected taxonomic distance over retained k-mer-similar pairs)
- Linear regression of error rate on confusion index with Pearson r2 and P-value (target: r2 = 0.72, P = 1.4e-4)
- confusion_index_vs_error.csv containing confusion indices, error rates, and correlation statistics

## How to apply

Start from the per-habitat bespoke taxonomic weights generated with q2-clawback's generate_class_weights for each evaluated environment. From each habitat's weighted reference distribution, sample 10^8 pairs of reference sequences and retain only pairs whose k-mer Bray-Curtis dissimilarity is < 0.25, i.e., pairs a naive Bayes classifier could plausibly confuse. For the retained pairs, compute the confusion index as the log of the expected taxonomic distance, measuring distance between taxonomies with the Greengenes taxonomy. Join these per-habitat confusion indices with the per-habitat bespoke species-level error rates measured on held-out test samples, fit a linear regression of error rate on confusion index, and report Pearson r2 and its P-value; the reported result to match is r2 = 0.72 (P = 1.4 × 10−4, Fig. 5). Write the confusion indices, error rates, and correlation statistics to confusion_index_vs_error.csv.

## Related tools

- **q2-clawback** (Generates the per-habitat bespoke taxonomic weights whose weighted reference distribution is sampled to estimate the confusion index) — https://github.com/BenKaehler/q2-clawback
- **QIIME 2** (Plugin framework under which q2-clawback weights and q2-feature-classifier classification (and thus the per-habitat error rates) are produced)
- **q2-feature-classifier** (Produces the bespoke species-level classifications whose per-habitat error rates are correlated against the confusion index) — https://github.com/qiime2/q2-feature-classifier/
- **scikit-learn** (Used to fit the linear regression and compute the Pearson r2 and P-value between confusion index and error rate)
- **redbiom** (Fetches the habitat-specific (e.g., EMPO 3) samples from Qiita used to assemble the bespoke weights) — biocore/redbiom

## Evaluation signals

- Reproduced Pearson correlation between confusion index and per-habitat bespoke species-level error rate approximates r2 = 0.72 with P ≈ 1.4 × 10−4 (Fig. 5).
- Pair sampling used exactly 10^8 pairs drawn from each habitat's weighted reference distribution, and only pairs with k-mer Bray-Curtis dissimilarity < 0.25 are retained for the index.
- Confusion index values are computed as log of expected taxonomic distance with distances measured in the Greengenes taxonomy.
- Every habitat in confusion_index_vs_error.csv has both a confusion index and a matched per-habitat bespoke error rate from the held-out test samples.
- Regression output is a linear fit of error rate on confusion index reporting both r2 and P-value, not just a scatter plot.

## Limitations

- Bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation), which the confusion index does not capture.
- The use of empirical species distributions introduces error sources such as misannotation of sample types or sequencing biases that propagate into the taxonomic weights.
- Systematic errors from bias present in common sequencing techniques are not controlled unless multiple technologies are integrated for microbial distribution estimation.
- The confusion index explains classification accuracy 'largely' but not entirely (r2 = 0.72, not 1.0), so residual error-rate variation remains unexplained.

## Evidence

- [abstract] We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated.: "We demonstrate that classification accuracy degrades linearly with the degree to which that assumption is violated, and in practice it is always violated."
- [discussion] demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy: "demonstrated that the more specific the taxonomic weights are to a query sample’s environment, the better the classification accuracy"
- [discussion] bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation): "bespoke classification is not immune to errors that result from poorly curated reference data (e.g., reference sequence misannotation)"
- [readme] Ready-made taxonomic weights generated by q2-clawback for use with the q2-feature-classifier taxonomy classifier.: [evidence span withheld — anchor status: external_doc]
