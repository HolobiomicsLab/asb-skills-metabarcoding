---
name: mock-community-profiler-performance-statistics
description: Use when per-sample benchmark metrics (sensitivity, specificity, log
  NRMSE, alpha diversity deviations) exist for multiple profiler-library pairings
  on the same mock community samples and you must determine which pairings significantly
  outperform others.
when_to_use_negative:
- Input is raw sequencing reads rather than computed per-sample benchmark metrics
  — pipeline profiling (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) must be run
  first
- Samples lack a ground-truth mock community composition, so sensitivity, specificity,
  and NRMSE cannot be computed against expected abundances
- There is only a single profiler-library pairing, since pairwise contrasts with Tukey
  correction require multiple pairings
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0622
tools:
- name: lme4
  role: Fits the linear mixed-effects model metric ~ pairing + (1|sample) for each
    benchmark metric via lmer()
- name: lsmeans
  role: Computes estimated marginal means per pairing and Tukey-adjusted pairwise
    contrast p-values
- name: DADA2
  role: One of the evaluated 16S profiling tools whose per-sample benchmark metrics
    feed the mixed models
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:b690a47b9864e8ce3e3b76cfaf5b3b80d21bab95123f098dc56a87b57a84123f
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
n_claims: 7
---

# mock-community-profiler-performance-statistics

## Summary

Statistical skill for benchmarking 16S profiler/reference-library pairings using linear mixed-effects models with Tukey-corrected post hoc comparisons, as used to rank DADA2, QIIME 2, Mothur, PathoScope 2, and Kraken 2 on 136 publicly available mock community samples.

## When to use

Use when per-sample benchmark metrics (sensitivity, specificity, log NRMSE, alpha diversity deviations) exist for multiple profiler-library pairings on the same mock community samples and you must determine which pairings significantly outperform others.

## When NOT to use

- Input is raw sequencing reads rather than computed per-sample benchmark metrics — pipeline profiling (DADA2, QIIME 2, Mothur, PathoScope 2, Kraken 2) must be run first
- Samples lack a ground-truth mock community composition, so sensitivity, specificity, and NRMSE cannot be computed against expected abundances
- There is only a single profiler-library pairing, since pairwise contrasts with Tukey correction require multiple pairings

## Inputs

- Per-sample benchmark metric tables (sensitivity, specificity, log NRMSE, alpha diversity deviations) for all profiler-reference-library pairings
- 136 publicly available 16S mock community samples with varied species richness and evenness

## Outputs

- Per-metric estimated marginal means for each profiler-library pairing
- Pairwise contrast tables with Tukey-adjusted p-values matching Supplementary Tables S2-S7 layout
- lmm_posthoc_results.csv containing fitted-model summaries and post hoc comparison tables

## How to apply

Load the per-sample benchmark metric tables for all profiler-reference-library pairings across the 136 mock community samples. For each metric, fit a linear mixed-effects model with lme4::lmer() using the specification metric ~ pairing + (1|sample), treating the profiler-library pairing as the fixed effect and mock community sample as a random intercept to account for repeated measurements of the same community. Compute estimated marginal means for each pairing with lsmeans(), then run pairwise contrast comparisons with Tukey adjustment to obtain all pairwise differences, standard errors, and adjusted p-values. Assemble the estimated marginal means, contrasts, and Tukey-adjusted p-values into per-metric comparison tables matching the layout of Supplementary Tables S2-S7, and save model summaries and post hoc tables to lmm_posthoc_results.csv. Validation: confirm that Kraken 2 + Greengenes alpha diversity deviations are significant at p < 0.001 versus all other pairings, and that at the species level Kraken 2 paired with its Standard database and with SILVA has the lowest log NRMDE/NRMSE per Table S7 pairwise p-values.

## Related tools

- **lme4** (Fits the linear mixed-effects model metric ~ pairing + (1|sample) for each benchmark metric via lmer())
- **lsmeans** (Computes estimated marginal means per pairing and Tukey-adjusted pairwise contrast p-values)
- **DADA2** (One of the evaluated 16S profiling tools whose per-sample benchmark metrics feed the mixed models)

## Examples

```
fit <- lme4::lmer(metric ~ pairing + (1|sample), data=metrics); lsmeans::lsmeans(fit, pairwise ~ pairing, adjust="tukey")
```

## Evaluation signals

- Random-intercept term (1|sample) is present in every fitted lmer() model, with pairing as fixed effect
- All pairwise p-values are Tukey-adjusted, and the significance pattern matches Supplementary Tables S2-S7
- Kraken 2 + Greengenes alpha diversity deviations are significant at p < 0.001 versus all other pairings
- At the species level, Kraken 2 with its Standard database and with SILVA shows the lowest log NRMSE, better than all other methods per Table S7
- All fitted-model summaries and post hoc tables are written to lmm_posthoc_results.csv

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors (pipetting variation, species lysis differences, PCR amplification bias, contamination) are introduced at every experimental step, so no pipeline can perfectly measure a mock community
- Benchmark conclusions are specific to 16S amplicon data and the reference library versions tested (e.g. Greengenes 13_8, SILVA 138.1, RefSeq 2018/2020, Kraken 2 Standard downloaded 8/20/2020)
- RefSeq libraries retrieved from public repositories required additional filtering and correction for contamination, which can bias downstream accuracy comparisons

## Evidence

- [methods] For each metric, fit a linear mixed-effects model: [evidence span withheld — anchor status: not_found]
- [methods] Tukey-adjusted post hoc comparisons: [evidence span withheld — anchor status: not_found]
- [abstract] 136 mock community samples: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [discussion] Ground truth limitation: "provide a semblance of a “ground truth” to assess 16S amplicon sequencing analysis methods"
- [readme] Repository code for library construction: "Kraken 2 Greengenes indices were obtained by running the following code"
