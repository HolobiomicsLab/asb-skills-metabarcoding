---
name: poisson-glm-richness-modeling
description: Use when per-location count data (e.g. MOTU richness or sequencing read
  abundance) must be compared across a categorical sampling factor such as geographic
  location, and pairwise post-hoc contrasts are needed.
when_to_use_negative:
- The response variable is a non-count diversity index (e.g. Shannon) rather than
  MOTU richness or raw read counts, where Poisson assumptions do not hold
- The design involves covariates or continuous predictors rather than a single categorical
  location factor for Tukey contrasts
- Input is the raw FASTQ/MOTU table rather than the aggregated per-location richness
  and read abundance table
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: sumaclust
  role: Delimits MOTUs (99% similarity threshold) whose per-location richness feeds
    the GLM response variable
  repo: http://github.com/metabarpark
- name: OBITools
  role: Bioinformatic suite producing the MOTU table from which per-location richness
    and read abundances are tabulated
  repo: http://github.com/metabarpark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:636832137089415015d3c1e346bfb40eec9187344c649586592a87fea94c9680
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - sumaclust
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# poisson-glm-richness-modeling

## Summary

Fit a Poisson generalized linear model with location as a categorical factor to test whether elasmobranch MOTU richness and square-root-transformed eDNA read abundance differ among sampling locations, followed by post-hoc Tukey pairwise contrasts. This reproduces the statistical test used by Bakker et al. (2017) to link shark diversity patterns to anthropogenic pressure levels.

## When to use

Use when per-location count data (e.g. MOTU richness or sequencing read abundance) must be compared across a categorical sampling factor such as geographic location, and pairwise post-hoc contrasts are needed.

## When NOT to use

- The response variable is a non-count diversity index (e.g. Shannon) rather than MOTU richness or raw read counts, where Poisson assumptions do not hold
- The design involves covariates or continuous predictors rather than a single categorical location factor for Tukey contrasts
- Input is the raw FASTQ/MOTU table rather than the aggregated per-location richness and read abundance table

## Inputs

- Per-location MOTU richness table (counts of MOTUs per sampling location)
- Per-location eDNA sequence read abundance table (from the OBITools/sumaclust MOTU table)
- Location metadata assigning samples to Caribbean and New Caledonia sampling sites

## Outputs

- GLM model summaries (Poisson) for MOTU richness and square-root-transformed read abundance
- Post-hoc Tukey pairwise contrast tables with P-values (e.g. Bahamas vs Jamaica P=0.039, Chesterfield vs Noumea P=0.013)
- glm_tukey_results.csv combining model summaries and Tukey contrast tables

## How to apply

Load the per-location MOTU richness and read abundance table derived from the metabarcoding MOTU table produced by the OBITools/sumaclust pipeline (99% similarity threshold, single-read MOTUs removed) and the custom R scripts from github.com/metabarpark. Square-root transform the sequence read abundances as specified for the abundance response variable, while MOTU richness is used untransformed. Fit a Poisson GLM with the glm2 package, entering location as a categorical factor, run separately for richness and abundance. Apply post-hoc Tukey comparisons with the multcomp package to obtain all pairwise contrasts among locations (e.g. Bahamas vs Jamaica, Chesterfield vs Noumea), and extract the pairwise P-values.

## Related tools

- **sumaclust** (Delimits MOTUs (99% similarity threshold) whose per-location richness feeds the GLM response variable) — http://github.com/metabarpark
- **OBITools** (Bioinformatic suite producing the MOTU table from which per-location richness and read abundances are tabulated) — http://github.com/metabarpark

## Examples

```
fit <- glm2(richness ~ location, family=poisson, data=motu_loc); summary(glht(fit, linfct=mcp(location="Tukey")))
```

## Evaluation signals

- Bahamas vs Jamaica richness contrast yields P = 0.039 and Bahamas vs Belize P = 0.095
- Chesterfield vs Noumea richness contrast yields P = 0.013; Bahamas vs other Caribbean sites P < 0.001
- Turks & Caicos contrasts give P = 0.033 (vs Jamaica) and P = 0.047 (vs Belize); New Caledonia abundance comparisons give P < 1e-10
- Abundance model was fit on square-root-transformed read counts, not raw counts
- glm_tukey_results.csv contains both model summaries and the full Tukey contrast tables for both response variables

## Limitations

- Single-read MOTUs are removed from the upstream MOTU table, but contamination or sequencing error cannot be completely excluded as their cause, so richness counts may include false positives if not filtered
- eDNA read abundance is not empirically verified to reflect community biomass for elasmobranchs, so abundance-model contrasts are interpreted cautiously
- Primer bias and species-specific shedding rates may limit quantitative interpretation of read counts across locations

## Evidence

- [discussion] A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location: "A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location"
- [discussion] All custom R scripts are publicly available from http://github.com/metabarpark.: "All custom R scripts are publicly available from http://github.com/metabarpark."
- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance: "Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance"
