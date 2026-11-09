---
name: benjamini-hochberg-fdr-multiple-testing-correction
description: Use when a statistical test (e.g., a linear mixed model with diet as
  fixed effect) has been run independently for many taxa or features, such as hundreds
  of SGBs, and raw p-values must be converted into BH-adjusted FDR values before declaring
  biomarkers.
when_to_use_negative:
- There is only a single hypothesis test, so no multiple-testing correction is needed.
- The analysis requires strict control of family-wise error rate (e.g., confirmatory
  claims) — BH controls expected false discoveries, not the probability of any false
  positive; a Bonferroni-style correction would be the wrong-tool counterpart to reach
  for instead.
- The p-values are not a valid test family (e.g., they come from different nested
  test stages or dependent subsets), which can distort BH calibration.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: statsmodels
  role: Provides MixedLM for the per-SGB diet association models whose p-values are
    then BH-corrected (statsmodels.stats.multitest implements fdr_bh); named as an
    implementation option in the source workflow.
- name: MetaPhlAn 4
  role: Upstream profiler generating the SGB-level relative abundance table (including
    uSGBs) whose per-SGB test p-values are the family corrected by BH FDR.
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:454343f1cb080a42a08e935a3f5f4d11190f026bd90644d5b3886009fac056e4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - statsmodels
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 3
---

# benjamini-hochberg-fdr-multiple-testing-correction

## Summary

Applies Benjamini-Hochberg false-discovery-rate correction to the per-taxon association p-values produced by high-dimensional testing (here, per-SGB linear mixed models of diet), so that a controlled set of significant biomarkers can be reported. In the source study this step gates which MetaPhlAn 4 SGBs are called high-fat-diet biomarkers at FDR < 0.2.

## When to use

Use when a statistical test (e.g., a linear mixed model with diet as fixed effect) has been run independently for many taxa or features, such as hundreds of SGBs, and raw p-values must be converted into BH-adjusted FDR values before declaring biomarkers.

## When NOT to use

- There is only a single hypothesis test, so no multiple-testing correction is needed.
- The analysis requires strict control of family-wise error rate (e.g., confirmatory claims) — BH controls expected false discoveries, not the probability of any false positive; a Bonferroni-style correction would be the wrong-tool counterpart to reach for instead.
- The p-values are not a valid test family (e.g., they come from different nested test stages or dependent subsets), which can distort BH calibration.

## Inputs

- Vector of raw association p-values, one per SGB, from per-taxon linear mixed models (diet fixed effect, sex/age/genetic background/vendor covariates)
- Corresponding SGB identifiers and per-SGB effect sizes (to be carried through to the final biomarker table)

## Outputs

- BH-adjusted FDR (q-value) per SGB
- List of significant SGBs retained at FDR < 0.2
- diet_biomarkers_sgb.tsv: table of surviving SGB biomarkers with effect sizes, p-values and FDR values

## How to apply

Collect the raw p-values from the per-SGB tests — in the source workflow, a random-intercept linear mixed model (lme4 or statsmodels MixedLM) per SGB with diet (HF vs chow) as fixed effect and sex, age, genetic background and vendor as covariates, fitted on CLR-transformed MetaPhlAn 4 SGB abundances filtered to mean relative abundance > 1%. Run the Benjamini-Hochberg step-up procedure over the whole family of per-SGB p-values (e.g., statsmodels.stats.multitest.multipletests with method='fdr_bh') rather than correcting each model separately. Retain SGBs whose BH-adjusted FDR is < 0.2 — the study's deliberately lenient threshold appropriate for exploratory biomarker discovery. Attach effect sizes, raw p-values and FDR values for the surviving SGBs to the output table; in the source cohort this yields 18 SGB biomarkers, 13 of them uSGBs (72%), including Lachnospiraceae bacterium 28_4 SGB7272, Lactobacillus johnsonii SGB7041 and Faecalibaculum rodentium SGB4047.

## Related tools

- **statsmodels** (Provides MixedLM for the per-SGB diet association models whose p-values are then BH-corrected (statsmodels.stats.multitest implements fdr_bh); named as an implementation option in the source workflow.)
- **MetaPhlAn 4** (Upstream profiler generating the SGB-level relative abundance table (including uSGBs) whose per-SGB test p-values are the family corrected by BH FDR.) — https://github.com/biobakery/MetaPhlAn

## Examples

```
from statsmodels.stats.multitest import multipletests; rej, fdr, _, _ = multipletests(pvals_sgb, alpha=0.2, method='fdr_bh')
```

## Evaluation signals

- Output table contains exactly the reported 18 SGB biomarkers called at FDR < 0.2 and mean abundance > 1%.
- Approximately 72% of retained biomarkers (13 of 18) are classified as uSGBs.
- The three anchor SGBs — Lachnospiraceae bacterium 28_4 SGB7272, Lactobacillus johnsonii SGB7041 and Faecalibaculum rodentium SGB4047 — are present in the table.
- BH-adjusted FDR values are monotone non-decreasing with raw p-values and all lie in [0, 1].
- Every reported FDR is < 0.2 and every retained SGB passes the >1% mean relative abundance filter.

## Limitations

- The FDR < 0.2 threshold used in the source study is lenient, so surviving biomarkers are exploratory candidates and the uSGB majority (72%) should be interpreted with that caveat.
- BH assumes the p-value family is reasonably behaved; heavily correlated per-taxon tests (e.g., related SGBs sharing a clade) can affect calibration of adjusted q-values.
- MetaPhlAn 4's profiling itself remains incomplete for under-characterized habitats: 'Environmental, nonhost-associated, and other under-studied microbial communities are still highly enriched for sequences not captured even by current uSGBs'.

## Evidence

- [methods] Correct the per-SGB diet association p-values for multiple testing with Benjamini-Hochberg FDR and retain SGBs with FDR < 0.2.: [evidence span withheld — anchor status: not_found]
- [results] Most of the high-fat diet SGB biomarkers identified via linear mixed models at FDR < 0.2 were uncharacterized species: 13 uSGBs out of the 18 biomarkers (72%) were over-abundant on a hyper-caloric diet.: [evidence span withheld — anchor status: not_found]
- [abstract] Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes.: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
