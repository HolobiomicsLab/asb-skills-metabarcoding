---
name: tukey-corrected-posthoc-contrast-analysis
description: Use when comparing multiple profiler/library pairings on per-sample benchmark
  metrics and you need statistically corrected pairwise significance calls (e.g.,
  ranking Kraken 2 vs. DADA2/QIIME 2/Mothur/PathoScope on 136 mock community samples).
when_to_use_negative:
- When the input is already a table of pairwise contrasts or significance calls —
  no model fitting is needed
- When the analysis goal is raw per-sample metric computation (sensitivity/specificity/NRMSE
  per sample) rather than statistical comparison of pairings
- When observations are not nested by a shared unit (e.g., no repeated measurement
  of the same mock community sample), in which case a random intercept of sample is
  not warranted
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: lme4
  role: Fits the linear mixed-effects model metric ~ pairing + (1|sample) via lmer()
    for each benchmark metric
- name: lsmeans
  role: Computes estimated marginal means per pairing and performs Tukey-adjusted
    pairwise contrast comparisons
- name: DADA2
  role: One of the 16S profiling pipelines whose outputs (per-sample benchmark metrics
    against reference libraries) are compared by the LMM/post hoc workflow
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:bea69f6172369ef5eecafd8bb8bdfc7b2e478582b68b3c2d7cdaa83bc5a1345f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - lme4
  - lsmeans
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tukey-corrected-posthoc-contrast-analysis

## Summary

Fits linear mixed-effects models to per-sample benchmark metrics (sensitivity, specificity, NRMSE, alpha diversity) across profiler/reference-library pairings and extracts all pairwise Tukey-adjusted post hoc contrasts, reproducing the significance tables used to rank 16S taxonomic profilers.

## When to use

Use when comparing multiple profiler/library pairings on per-sample benchmark metrics and you need statistically corrected pairwise significance calls (e.g., ranking Kraken 2 vs. DADA2/QIIME 2/Mothur/PathoScope on 136 mock community samples).

## When NOT to use

- When the input is already a table of pairwise contrasts or significance calls — no model fitting is needed
- When the analysis goal is raw per-sample metric computation (sensitivity/specificity/NRMSE per sample) rather than statistical comparison of pairings
- When observations are not nested by a shared unit (e.g., no repeated measurement of the same mock community sample), in which case a random intercept of sample is not warranted

## Inputs

- Per-sample benchmark metric tables (sensitivity, specificity, NRMSE/log NRMSE, alpha diversity estimates) for the 136 mock community samples
- Profiler/reference-library pairing labels for each sample-metric observation (e.g., Kraken 2 + Standard, Kraken 2 + SILVA, PathoScope + Greengenes)

## Outputs

- Estimated marginal means per profiler-library pairing for each metric
- Pairwise contrast tables with differences, standard errors, and Tukey-adjusted p-values matching Supplementary Tables S2-S7
- Fitted-model summaries and post hoc comparison tables (e.g., lmm_posthoc_results.csv)

## How to apply

Load per-sample metric tables (sensitivity, specificity, log NRMSE, alpha diversity deviations) for the 136 publicly available mock community samples spanning all profiler-reference-library pairings. For each metric, fit a linear mixed-effects model with lme4::lmer() using the specification metric ~ pairing + (1|sample), treating the profiler-library pairing as the fixed effect and the mock community sample as a random intercept to account for repeated measurement of the same community. Compute estimated marginal means for each pairing with lsmeans(), then run lsmeans() pairwise contrasts with Tukey adjustment to obtain all pairwise differences, standard errors, and adjusted p-values. Assemble the estimated marginal means, pairwise contrasts, and Tukey-adjusted p-values into per-metric comparison tables matching the layout of Supplementary Tables S2-S7 and save them (e.g., to lmm_posthoc_results.csv). Validate against known patterns, such as Kraken 2 having the lowest log NRMSE at the species level when paired with its Standard database and with SILVA, and Kraken 2 + Greengenes alpha diversity deviations being significant at p < 0.001 versus all other pairings.

## Related tools

- **lme4** (Fits the linear mixed-effects model metric ~ pairing + (1|sample) via lmer() for each benchmark metric)
- **lsmeans** (Computes estimated marginal means per pairing and performs Tukey-adjusted pairwise contrast comparisons)
- **DADA2** (One of the 16S profiling pipelines whose outputs (per-sample benchmark metrics against reference libraries) are compared by the LMM/post hoc workflow)

## Examples

```
fit <- lme4::lmer(nrmse ~ pairing + (1|sample), data = metrics); lsmeans::lsmeans(fit, pairwise ~ pairing, adjust = "tukey")
```

## Evaluation signals

- Kraken 2 + Greengenes alpha diversity deviations are significant at p < 0.001 versus all other pairings, matching the reported pairwise pattern
- At the species level, Kraken 2 has the lowest log NRMSE when paired with its Standard database and with SILVA, per Table S7 pairwise p-values
- All pairwise contrasts include Tukey-adjusted p-values, standard errors, and estimated marginal means assembled per metric as in Supplementary Tables S2-S7
- Each fitted model uses sample as a random intercept and pairing as the sole fixed effect (metric ~ pairing + (1|sample))

## Limitations

- Mock communities provide only a semblance of a ground truth; technical bias and errors are introduced at every experimental step (pipetting variations, lysis difficulty, amplification bias, reagent contamination), so significance differences reflect performance under these biases
- Results apply to 16S amplicon mock community benchmarking of genus- and species-level assignment; conclusions may not transfer to shotgun metagenomic data
- Significance patterns depend on the specific reference library versions used (e.g., Greengenes 13_8, SILVA 138.1, Kraken Standard downloaded 8/20/2020); newer library versions may change rankings

## Evidence

- [other] For each metric, fit a linear mixed-effects model with lme4::lmer() using the specification metric ~ pairing + (1|sample): [evidence span withheld — anchor status: not_found]
- [other] Run lsmeans() pairwise contrast comparisons with Tukey adjustment: [evidence span withheld — anchor status: not_found]
- [results] Kraken 2 had the lowest log NRMSE when paired with its Standard database and with SILVA: [evidence span withheld — anchor status: not_found]
- [abstract] publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [readme] Kraken 2 Greengenes indices were obtained by running the following code: "kraken2-build --db ${DBNAME} --special greengenes --threads 1"
