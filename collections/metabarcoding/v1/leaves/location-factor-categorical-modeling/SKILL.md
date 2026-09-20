---
name: location-factor-categorical-modeling
description: Use when per-location count data (e.g. MOTU richness or read abundances
  from metabarcoding) must be tested for significant differences among sampling locations,
  and pairwise location contrasts with P-values are needed.
when_to_use_negative:
- When the response variable is continuous and approximately normal rather than count
  data (Poisson GLM and square-root-transformed read counts are specific to this design)
- When the goal is taxonomic assignment or MOTU delimitation itself — this skill operates
  downstream on the already-constructed MOTU richness/abundance table
- When samples lack a categorical grouping factor (e.g. single-site time series),
  since location as a factor is the core of the model
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3058
tools:
- name: glm2
  role: Fits the Poisson generalized linear model with location as a categorical factor,
    separately for MOTU richness and square-root-transformed read abundance
- name: multcomp
  role: R package used to run post-hoc Tukey comparisons yielding pairwise location
    contrasts and P-values
- name: custom R scripts (metabarpark)
  role: Publicly available R scripts used to produce the per-location richness/abundance
    tables and run the GLM/Tukey analysis
  repo: github.com/metabarpark
- name: sumaclust
  role: Upstream MOTU delimitation (99% similarity threshold) that produces the MOTU
    table from which the per-location response variables are derived
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:e85e7b33d403b83ca0f6cb6e6d82cd9bda069ab1402a630abbef614debd351f6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/metabarpark
  tools:
  - custom R scripts (metabarpark)
  - glm2
  - multcomp
  - sumaclust
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3058
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# location-factor-categorical-modeling

## Summary

Fits generalized linear models with sampling location entered as a categorical factor to test whether MOTU richness and square-root-transformed eDNA read abundance differ among sites, followed by post-hoc Tukey pairwise contrasts (multcomp) to identify which locations differ.

## When to use

Use when per-location count data (e.g. MOTU richness or read abundances from metabarcoding) must be tested for significant differences among sampling locations, and pairwise location contrasts with P-values are needed.

## When NOT to use

- When the response variable is continuous and approximately normal rather than count data (Poisson GLM and square-root-transformed read counts are specific to this design)
- When the goal is taxonomic assignment or MOTU delimitation itself — this skill operates downstream on the already-constructed MOTU richness/abundance table
- When samples lack a categorical grouping factor (e.g. single-site time series), since location as a factor is the core of the model

## Inputs

- Per-location MOTU richness table derived from the OBITools/sumaclust metabarcoding MOTU table
- Per-location eDNA sequence read abundance table (counts per location)
- Location labels for each sample (categorical factor levels: e.g. Bahamas, Belize, Jamaica, Turks & Caicos, Chesterfield, Noumea)

## Outputs

- GLM model summaries for each response variable (Poisson glm2 fits with location as categorical factor)
- Post-hoc Tukey pairwise contrast table with P-values (from multcomp)
- glm_tukey_results.csv containing model summaries and Tukey contrast tables

## How to apply

Load the per-location MOTU richness and read abundance table derived from the metabarcoding MOTU table produced by the OBITools/sumaclust pipeline (99% similarity threshold, single-read MOTUs removed) into R. Square-root transform the sequence read abundances, as specified for the abundance response variable, while MOTU richness is used as a count response. Fit a Poisson generalized linear model using glm2 with location entered as a categorical factor, running the model separately for MOTU richness and for square-root-transformed read abundance. Apply post-hoc Tukey comparisons with the multcomp package to obtain pairwise contrasts among sampling locations for each response variable, then extract P-values and compare them against the reported contrasts (Bahamas vs Jamaica P=0.039, Bahamas vs Belize P=0.095, Chesterfield vs Noumea P=0.013, Bahamas vs other Caribbean P<0.001, Turks & Caicos vs Jamaica P=0.033 and vs Belize P=0.047, New Caledonia abundance comparisons P<1e-10). Write model summaries and Tukey contrast tables to a results CSV.

## Related tools

- **glm2** (Fits the Poisson generalized linear model with location as a categorical factor, separately for MOTU richness and square-root-transformed read abundance)
- **multcomp** (R package used to run post-hoc Tukey comparisons yielding pairwise location contrasts and P-values)
- **custom R scripts (metabarpark)** (Publicly available R scripts used to produce the per-location richness/abundance tables and run the GLM/Tukey analysis) — github.com/metabarpark
- **sumaclust** (Upstream MOTU delimitation (99% similarity threshold) that produces the MOTU table from which the per-location response variables are derived)

## Examples

```
Rscript -e 'fit <- glm2(richness ~ location, family=poisson, data=motu_by_location); summary(multcomp::glht(fit, linfct=mcp(location="Tukey")))'
```

## Evaluation signals

- Reproduced Tukey P-values match reported values: Bahamas vs Jamaica P=0.039, Bahamas vs Belize P=0.095, Chesterfield vs Noumea P=0.013
- Bahamas vs other Caribbean locations give P<0.001 for MOTU richness; Turks & Caicos vs Jamaica P=0.033 and vs Belize P=0.047
- New Caledonia read abundance comparisons yield P<1e-10
- Read abundances are square-root transformed before modeling; richness is modeled as a Poisson count response
- glm_tukey_results.csv contains both model summaries and the full Tukey contrast table

## Limitations

- The relationship between eDNA read abundance and community biomass has not been empirically verified for elasmobranchs, so quantitative interpretation of the abundance response remains ambiguous
- Shedding rates and primer bias may differ among species/individuals (primer efficiency may vary among species templates), limiting quantitative interpretation of read counts
- No studies had published evidence of a relationship between relative species abundance within a community and eDNA read abundance at the time of this study

## Evidence

- [methods] A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location: "A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location"
- [methods] All custom R scripts are publicly available from http://github.com/metabarpark.: "All custom R scripts are publicly available from http://github.com/metabarpark."
- [methods] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] While currently no experiments have been performed to empirically verify the relationship between read abundance and community biomass for elasmobranchs in particular: [evidence span withheld — anchor status: not_found]
- [discussion] Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance among our samples: "Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance among our samples"
