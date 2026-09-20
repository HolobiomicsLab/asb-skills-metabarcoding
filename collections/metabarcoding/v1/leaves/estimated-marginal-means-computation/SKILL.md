---
name: estimated-marginal-means-computation
description: Use when a fitted linear mixed-effects model (e.g. metric ~ pairing +
  (1|sample)) needs post hoc pairwise comparisons between categorical factor levels,
  such as profiler-library pairings across 136 mock community samples.
when_to_use_negative:
- The input is raw per-sample metric data with no fitted mixed-effects model — fit
  the lmer() model first; EMMs cannot be computed without a model object
- The comparison involves only two groups or pre-specified orthogonal contrasts where
  Tukey's all-pairwise adjustment is unnecessary and overly conservative
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_2269
tools:
- name: lme4
  role: Fits the linear mixed-effects model (lmer) with pairing as fixed effect and
    mock community sample as random intercept, prior to EMM computation
- name: lsmeans
  role: Computes estimated marginal means for each pairing and performs Tukey-adjusted
    pairwise contrast comparisons on the fitted lmer models
- name: DADA2
  role: One of the 16S profiling pipelines whose per-sample benchmark metrics feed
    the mixed models and post hoc EMM comparisons
  repo: github.com/aubreyodom/16SBenchmarking
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:392e3b991842553013fb482a809be9caca8abfe3a8bc7c943be2a793f672f726
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - lme4
  - lsmeans
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# estimated-marginal-means-computation

## Summary

Compute estimated marginal means (EMMs) for each profiler/reference-library pairing from fitted linear mixed-effects models, then derive Tukey-adjusted pairwise contrasts, replicating the per-metric comparison tables (Supplementary Tables S2-S7) of the 16S benchmarking study.

## When to use

Use when a fitted linear mixed-effects model (e.g. metric ~ pairing + (1|sample)) needs post hoc pairwise comparisons between categorical factor levels, such as profiler-library pairings across 136 mock community samples.

## When NOT to use

- The input is raw per-sample metric data with no fitted mixed-effects model — fit the lmer() model first; EMMs cannot be computed without a model object
- The comparison involves only two groups or pre-specified orthogonal contrasts where Tukey's all-pairwise adjustment is unnecessary and overly conservative

## Inputs

- Fitted linear mixed-effects model object from lme4::lmer() with specification metric ~ pairing + (1|sample)
- Per-sample benchmark metric tables (sensitivity, specificity, log NRMSE, alpha diversity) for the 136 publicly available mock community samples across all profiler-reference-library pairings

## Outputs

- Estimated marginal means per profiler-library pairing for each metric
- Pairwise contrast tables with differences, standard errors, and Tukey-adjusted p-values matching Supplementary Tables S2-S7
- lmm_posthoc_results.csv containing fitted-model summaries and post hoc comparison tables

## How to apply

After fitting an lme4::lmer() model such as metric ~ pairing + (1|sample) — with the profiler-reference-library pairing as fixed effect and mock community sample as random intercept — call lsmeans() on the fitted model to obtain estimated marginal means for each pairing. Then run lsmeans() pairwise contrast comparisons with Tukey adjustment to get all pairwise differences, standard errors, and adjusted p-values between pairings. Assemble the EMMs, contrasts, and Tukey-adjusted p-values into per-metric comparison tables mirroring Supplementary Tables S2-S7 (sensitivity, specificity, NRMSE, and alpha diversity metrics), and save everything to lmm_posthoc_results.csv. The Tukey correction controls family-wise error across the many pairwise contrasts between all profiler-library pairings (e.g. Kraken 2 Standard, Kraken 2 SILVA, DADA2 Greengenes, etc.).

## Related tools

- **lme4** (Fits the linear mixed-effects model (lmer) with pairing as fixed effect and mock community sample as random intercept, prior to EMM computation)
- **lsmeans** (Computes estimated marginal means for each pairing and performs Tukey-adjusted pairwise contrast comparisons on the fitted lmer models)
- **DADA2** (One of the 16S profiling pipelines whose per-sample benchmark metrics feed the mixed models and post hoc EMM comparisons) — github.com/aubreyodom/16SBenchmarking

## Examples

```
library(lme4); library(lsmeans); fit <- lmer(metric ~ pairing + (1|sample), data = metrics); emm <- lsmeans(fit, pairwise ~ pairing, adjust = "tukey")
```

## Evaluation signals

- Kraken 2 + Greengenes alpha diversity deviations are significant at p < 0.001 versus all other pairings, matching the reported pairwise significance pattern in the supplementary tables
- At the species level, Kraken 2 shows the lowest log NRMDE for its Standard database and SILVA pairings and is significantly better than all other methods per Table S7 pairwise p-values
- The output tables reproduce the layout of Supplementary Tables S2-S7 with estimated marginal means, pairwise contrasts, and Tukey-adjusted p-values per metric
- lmm_posthoc_results.csv contains fitted-model summaries plus post hoc comparison tables for all four metrics (sensitivity, specificity, NRMSE, alpha diversity)
- Every pairwise contrast between profiler-library pairings has an associated Tukey-adjusted p-value and standard error (no missing comparisons)

## Limitations

- Mock communities provide only a semblance of ground truth — technical bias and errors (pipetting variation, lysis bias, amplification bias, reagent contamination) are introduced at every experimental step, so EMM differences reflect estimates relative to imperfect references
- No analysis pipeline could theoretically exist to perfectly measure a mock community, so significant pairwise contrasts should not be interpreted as absolute accuracy
- The random-intercept structure (1|sample) assumes a single observation per sample-pairing cell; more complex designs would require a different mixed-model specification

## Evidence

- [other] lmer specification: [evidence span withheld — anchor status: not_found]
- [other] lsmeans workflow: [evidence span withheld — anchor status: not_found]
- [other] Tukey adjustment: [evidence span withheld — anchor status: not_found]
- [supplementary] validation pattern: [evidence span withheld — anchor status: not_found]
- [abstract] sample scope: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
