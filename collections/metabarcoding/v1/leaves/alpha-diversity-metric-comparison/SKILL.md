---
name: alpha-diversity-metric-comparison
description: Use when per-sample benchmark metrics (alpha diversity deviation, sensitivity,
  specificity, NRMSE) have been computed for multiple profiler-library pairings over
  repeated mock community samples and you must determine which pairings significantly
  outperform others.
when_to_use_negative:
- Input is raw 16S FASTQ data or ASV/OTU tables rather than already-computed per-sample
  benchmark metric tables — profiling and metric calculation must happen first
- Data are not repeated measures over shared samples (the (1|sample) random intercept
  and within-sample pairing comparison are inappropriate for independent single-observation
  designs)
- The goal is taxonomy assignment or database index construction (e.g., building Kraken
  2 Greengenes indices or custom PathoScope SILVA/RefSeq libraries) rather than statistical
  comparison of pairings
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3307
- http://edamontology.org/topic_3697
tools:
- name: lme4
  role: Fits the linear mixed-effects model metric ~ pairing + (1|sample) for each
    benchmark metric
- name: lsmeans
  role: Computes estimated marginal means and Tukey-adjusted pairwise contrasts between
    profiler-library pairings
- name: DADA2
  role: One of the evaluated 16S profiling pipelines whose per-pairing alpha diversity
    and accuracy metrics feed the LMM comparison
- name: Kraken 2
  role: Profiler whose pairings (e.g., with its Standard database, SILVA, Greengenes)
    are compared via the post hoc tables; Kraken 2 + Greengenes alpha diversity deviations
    are a key validation signal
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-023-40799-x
    title: Metagenomic profiling pipelines improve taxonomic classification for 16S
      amplicon sequencing data
  source_content_hash: sha256:1b5a4de6f760dee8a239bfd0a4dd368a9a53faec6a8a80a34b79af19b3575ac2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - DADA2
  - Kraken 2
  - lme4
  - lsmeans
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3307
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# alpha-diversity-metric-comparison

## Summary

Statistically compares alpha diversity estimates (and related benchmark metrics: sensitivity, specificity, log NRMSE) across profiler/reference-library pairings from 16S amplicon mock community benchmarks using linear mixed-effects models with Tukey-corrected pairwise contrasts, mirroring the Supplementary Tables S2-S7 analysis of the Faits and Odom-Mabey et al. Scientific Reports 2023 paper.

## When to use

Use when per-sample benchmark metrics (alpha diversity deviation, sensitivity, specificity, NRMSE) have been computed for multiple profiler-library pairings over repeated mock community samples and you must determine which pairings significantly outperform others.

## When NOT to use

- Input is raw 16S FASTQ data or ASV/OTU tables rather than already-computed per-sample benchmark metric tables — profiling and metric calculation must happen first
- Data are not repeated measures over shared samples (the (1|sample) random intercept and within-sample pairing comparison are inappropriate for independent single-observation designs)
- The goal is taxonomy assignment or database index construction (e.g., building Kraken 2 Greengenes indices or custom PathoScope SILVA/RefSeq libraries) rather than statistical comparison of pairings

## Inputs

- Per-sample benchmark metric tables (alpha diversity deviation, sensitivity, specificity, log NRMSE) for 136 publicly available mock community samples across all profiler-reference-library pairings
- Mapping of each sample-pairing observation to its profiler/library pairing and mock community sample ID

## Outputs

- Estimated marginal means per profiler-library pairing for each metric
- Pairwise contrast tables with differences, standard errors, and Tukey-adjusted p-values (Supplementary Tables S2-S7 layout)
- Fitted linear mixed-effects model summaries (lmer objects/summaries) and post hoc comparison tables, e.g., lmm_posthoc_results.csv

## How to apply

Load the per-sample metric tables for the 136 mock community samples spanning all profiler-reference-library pairings (e.g., Kraken 2, PathoScope 2, QIIME 2, Mothur, DADA2 crossed with Greengenes 13_8, SILVA 138, Kraken 2 Standard, RefSeq libraries). For each metric, fit a linear mixed-effects model with lme4::lmer() using the specification metric ~ pairing + (1|sample), treating the profiler-library pairing as the fixed effect and the mock community sample as a random intercept to account for repeated measurement of the same communities. Compute estimated marginal means for each pairing with lsmeans(), then run pairwise contrast comparisons with Tukey adjustment to obtain all pairwise differences, standard errors, and adjusted p-values. Assemble the estimated marginal means, pairwise contrasts, and Tukey-adjusted p-values into per-metric comparison tables matching the layout of Supplementary Tables S2-S7 and save model summaries plus post hoc tables (e.g., to lmm_posthoc_results.csv).

## Related tools

- **lme4** (Fits the linear mixed-effects model metric ~ pairing + (1|sample) for each benchmark metric)
- **lsmeans** (Computes estimated marginal means and Tukey-adjusted pairwise contrasts between profiler-library pairings)
- **DADA2** (One of the evaluated 16S profiling pipelines whose per-pairing alpha diversity and accuracy metrics feed the LMM comparison)
- **Kraken 2** (Profiler whose pairings (e.g., with its Standard database, SILVA, Greengenes) are compared via the post hoc tables; Kraken 2 + Greengenes alpha diversity deviations are a key validation signal)

## Examples

```
R: fit <- lme4::lmer(alpha_div ~ pairing + (1|sample), data = metrics); lsmeans::lsmeans(fit, pairwise ~ pairing, adjust = "tukey")
```

## Evaluation signals

- Kraken 2 + Greengenes alpha diversity deviations are significant at p < 0.001 versus all other pairings, matching the reported pairwise significance pattern in the supplementary tables
- Per-pairing species-level results reproduce Table S7 ordering, e.g., Kraken 2 paired with its Standard database and with SILVA having the lowest log NRMSE and beating all other methods
- Every one of the 136 mock community samples contributes one observation per pairing, and each model includes a random intercept per sample
- Number of pairwise contrasts equals all pairing combinations with Tukey adjustment applied to every p-value
- Output tables are laid out per metric matching Supplementary Tables S2-S7 (estimated marginal means, differences, standard errors, adjusted p-values)

## Limitations

- Mock communities provide only a semblance of ground truth; technical bias and errors are introduced at every experimental step, so no analysis pipeline could theoretically exist to perfectly measure a mock community
- Mock species relative abundances may be affected by subtle pipetting variations when aliquoting spike-in DNA, species vary in lysis difficulty, and amplification bias occurs during PCR despite universal primer design
- Contamination from reagents and local environment (air, gloves, pipette tip boxes) and reference-library contamination (e.g., E. coli 16S sequence in RefSeq T. flagellates scaffolds) can complicate results and downstream comparisons

## Evidence

- [methods] metric ~ pairing + (1|sample), treating the profiler-library pairing as the fixed effect and the mock community sample as a random intercept: [evidence span withheld — anchor status: not_found]
- [supplementary] Kraken 2 + Greengenes alpha diversity deviations significant at p < 0.001: [evidence span withheld — anchor status: not_found]
- [abstract] 136 mock community samples with varied richness and evenness: "publicly available mock community data from several sources, comprising 136 samples with varied species richness and evenness"
- [supplementary] Tukey-adjusted post hoc comparisons assemble per-metric tables S2-S7: [evidence span withheld — anchor status: not_found]
