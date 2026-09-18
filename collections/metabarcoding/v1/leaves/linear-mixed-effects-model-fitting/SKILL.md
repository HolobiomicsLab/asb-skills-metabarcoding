---
name: linear-mixed-effects-model-fitting
description: Use when per-sample benchmark metrics (sensitivity, specificity, NRMSE,
  alpha diversity) are collected across multiple profiler/reference-library pairings
  on the same 136 mock community samples, and you need pairwise significance testing
  between pairings while accounting for sample-to-sample
when_to_use_negative:
- The outcome is not a per-sample continuous metric with repeated measurements across
  pairings, so a random-intercept structure (1|sample) is inappropriate
- The data are count-based or binomial taxon detections that violate linear-model
  assumptions and would require a generalized mixed model rather than lmer()
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: lme4
  role: Fits the linear mixed-effects model metric ~ pairing + (1|sample) for each
    benchmark metric, with profiler-library pairing as the fixed effect and mock community
    sample as a random intercept
- name: lsmeans
  role: Computes estimated marginal means per pairing and performs Tukey-adjusted
    pairwise contrast comparisons to yield adjusted p-values for Supplementary Tables
    S2–S7
- name: DADA2
  role: One of the 16S profiling pipelines whose per-sample outputs (sensitivity,
    specificity, NRMSE, alpha diversity) feed the mixed-effects models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:9ab70a15b6f14d4c82f78ff33eacbbb07202db469054bf54aae075553ad9a551
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - lme4
  - lsmeans
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# linear-mixed-effects-model-fitting

## Summary

Fits linear mixed-effects models with lme4::lmer() to compare taxonomic profiler/reference-library pairings on per-sample benchmark metrics from 136 mock community 16S samples, then uses lsmeans() with Tukey adjustment for all pairwise post hoc contrasts as reported in Supplementary Tables S2–S7.

## When to use

Use when per-sample benchmark metrics (sensitivity, specificity, NRMSE, alpha diversity) are collected across multiple profiler/reference-library pairings on the same 136 mock community samples, and you need pairwise significance testing between pairings while accounting for sample-to-sample variation via random intercepts.

## When NOT to use

- The outcome is not a per-sample continuous metric with repeated measurements across pairings, so a random-intercept structure (1|sample) is inappropriate
- The data are count-based or binomial taxon detections that violate linear-model assumptions and would require a generalized mixed model rather than lmer()

## Inputs

- Per-sample benchmark metric tables (sensitivity, specificity, NRMSE/log NRMSE, alpha diversity) for 136 publicly available mock community samples
- All profiler/reference-library pairings (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2 x Greengenes, SILVA, RefSeq, Kraken 2 Standard)

## Outputs

- Per-metric comparison tables of estimated marginal means, pairwise contrasts, and Tukey-adjusted p-values matching Supplementary Tables S2–S7
- lmm_posthoc_results.csv with all fitted-model summaries and post hoc comparison tables

## How to apply

Load the per-sample benchmark metric tables (sensitivity, specificity, NRMSE, and alpha diversity) for the 136 publicly available mock community samples spanning all profiler-reference-library pairings. For each metric, fit a linear mixed-effects model with lme4::lmer() using the specification metric ~ pairing + (1|sample), treating the profiler-library pairing as the fixed effect and the mock community sample as a random intercept to control for repeated measurements of the same community. Compute estimated marginal means for each pairing with lsmeans() on each fitted model, then run lsmeans() pairwise contrast comparisons with Tukey adjustment to obtain all pairwise differences, standard errors, and adjusted p-values. Assemble the estimated marginal means, pairwise contrasts, and Tukey-adjusted p-values into per-metric comparison tables matching the layout of Supplementary Tables S2–S7, and save all fitted-model summaries and post hoc comparison tables to lmm_posthoc_results.csv.

## Related tools

- **lme4** (Fits the linear mixed-effects model metric ~ pairing + (1|sample) for each benchmark metric, with profiler-library pairing as the fixed effect and mock community sample as a random intercept)
- **lsmeans** (Computes estimated marginal means per pairing and performs Tukey-adjusted pairwise contrast comparisons to yield adjusted p-values for Supplementary Tables S2–S7)
- **DADA2** (One of the 16S profiling pipelines whose per-sample outputs (sensitivity, specificity, NRMSE, alpha diversity) feed the mixed-effects models)

## Examples

```
fit <- lme4::lmer(metric ~ pairing + (1|sample), data = metrics); lsmeans::lsmeans(fit, pairwise ~ pairing, adjust = "tukey")
```

## Evaluation signals

- Kraken 2 + Greengenes alpha diversity deviations are significant at p < 0.001 versus all other pairings, matching the reported pairwise significance pattern in the supplementary tables
- At the species level, Kraken 2 shows the lowest log NRMSE when paired with its Standard database and with SILVA, and is significantly better than all other methods per Table S7 pairwise p-values
- lmm_posthoc_results.csv contains fitted-model summaries plus estimated marginal means and Tukey-adjusted p-values for every metric (sensitivity, specificity, NRMSE, alpha diversity)
- Table layouts reproduce the structure of Supplementary Tables S2–S7 with all pairwise contrasts and adjusted p-values present
- Each fitted model includes a per-mock-community-sample random intercept rather than a simple fixed-effects-only linear model

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so modeled metrics absorb pipetting, lysis, PCR amplification, and contamination effects
- The linear (Gaussian) mixed-model formulation may be sensitive to non-normal metric distributions, and interpretation depends on the fixed set of 136 mock community samples
- Pairing significance reflects the specific reference-library versions benchmarked (Greengenes 13_8, SILVA 138.1, RefSeq 2018/2020, Kraken 2 Standard); newer library versions may change results

## Evidence

- [methods] fit a linear mixed-effects model with lme4::lmer() using the specification metric ~ pairing + (1|sample), treating the profiler-library pairing as the fixed effect: [evidence span withheld — anchor status: not_found]
- [methods] Run lsmeans() pairwise contrast comparisons with Tukey adjustment to obtain all pairwise differences, standard errors, and adjusted p-values between profiler-library pairings.: [evidence span withheld — anchor status: not_found]
- [supplementary] confirm that Kraken 2 + Greengenes alpha diversity deviations are significant at p < 0.001 versus all other pairings, matching the reported pairwise significance pattern: [evidence span withheld — anchor status: not_found]
- [supplementary] At the species level, Kraken 2 had the lowest log NRMSE when paired with its Standard database and with SILVA, and was better than all other methods, per the Table S7 pairwise comparison p-values.: [evidence span withheld — anchor status: not_found]
