---
name: ecological-diversity-hypothesis-testing
description: Use when per-location eDNA MOTU richness and sequence read abundance
  counts are available and the goal is to test whether these biodiversity metrics
  differ significantly among categorical sampling sites with a Poisson GLM and post-hoc
  Tukey contrasts.
when_to_use_negative:
- Input is raw sequencing reads or a sample-by-MOTU matrix — aggregate first via the
  OBITools/sumaclust pipeline (99% clustering, single-read MOTU removal) before applying
  this skill.
- The response variable is proportional or normalized (e.g., relative percentages)
  rather than counts — the Poisson family with square-root-transformed abundances
  assumed here would be misspecified.
- You need quantitative biomass estimates per species — the paper states no experiment
  has verified the read-abundance-to-biomass relationship for elasmobranchs, so abundance
  contrasts describe relative reads, not biomass.
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: glm2
  role: Fits the Poisson generalized linear model of MOTU richness and square-root-transformed
    read abundance with location as a categorical factor
- name: multcomp
  role: Runs the post-hoc Tukey pairwise contrasts among sampling locations for each
    response variable (glht/mcp)
- name: sumaclust
  role: Upstream MOTU delimitation at a constant 99% similarity threshold that produces
    the per-location richness and read-count table this skill tests
- name: custom R scripts (metabarpark)
  role: Publicly available R scripts implementing the GLM/Tukey analysis of the MOTU
    table
  repo: http://github.com/metabarpark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:4fc810fa468f1fdecd22e87ccf5160297b0a8013b8d74aa492f343b91a01600d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - custom R scripts (metabarpark)
  - glm2
  - multcomp
  - sumaclust
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# ecological-diversity-hypothesis-testing

## Summary

R-based hypothesis-testing skill for eDNA metabarcoding outputs: a Poisson glm2 model with sampling location as a categorical factor tests whether MOTU richness and square-root-transformed sequence read abundances differ among locations, with multcomp post-hoc Tukey contrasts quantifying every pairwise difference.

## When to use

Use when per-location eDNA MOTU richness and sequence read abundance counts are available and the goal is to test whether these biodiversity metrics differ significantly among categorical sampling sites with a Poisson GLM and post-hoc Tukey contrasts.

## When NOT to use

- Input is raw sequencing reads or a sample-by-MOTU matrix — aggregate first via the OBITools/sumaclust pipeline (99% clustering, single-read MOTU removal) before applying this skill.
- The response variable is proportional or normalized (e.g., relative percentages) rather than counts — the Poisson family with square-root-transformed abundances assumed here would be misspecified.
- You need quantitative biomass estimates per species — the paper states no experiment has verified the read-abundance-to-biomass relationship for elasmobranchs, so abundance contrasts describe relative reads, not biomass.

## Inputs

- Per-location MOTU richness counts derived from the OBITools/sumaclust metabarcoding MOTU table (99% similarity, single-read MOTUs removed)
- Per-location sequence read abundance counts (raw; square-root transformed before modeling)
- Sampling-location metadata mapping samples to categorical location levels (Caribbean: Bahamas/Bimini, Belize, Jamaica, Turks & Caicos; New Caledonia: Chesterfield, Noumea)

## Outputs

- Poisson GLM model summaries per response variable (glm2 fits)
- Post-hoc Tukey pairwise contrast tables with P-values (multcomp::glht)
- glm_tukey_results.csv containing model summaries and Tukey contrast tables for both response variables

## How to apply

Aggregate the OBITools/sumaclust MOTU table (99% similarity threshold, single-read MOTUs removed) into per-location response variables: MOTU richness as raw counts and sequence read abundance square-root transformed, as specified for the abundance response in the paper. In R, fit a Poisson generalized linear model using glm2 with location entered as a categorical factor, once for richness and once for transformed read abundance. Apply post-hoc Tukey comparisons with the multcomp package (glht with mcp(location='Tukey')) to obtain all pairwise contrasts among sampling locations. Validate the reproduction against the reported contrasts: Bahamas vs Jamaica P=0.039, Bahamas vs Belize P=0.095, Chesterfield vs Noumea P=0.013, Bahamas vs other Caribbean P<0.001, Turks & Caicos vs Jamaica P=0.033 and vs Belize P=0.047, and New Caledonia abundance comparisons P<1e-10. Write the model summaries and Tukey contrast tables to glm_tukey_results.csv; the custom R scripts are publicly available from http://github.com/metabarpark.

## Related tools

- **glm2** (Fits the Poisson generalized linear model of MOTU richness and square-root-transformed read abundance with location as a categorical factor)
- **multcomp** (Runs the post-hoc Tukey pairwise contrasts among sampling locations for each response variable (glht/mcp))
- **sumaclust** (Upstream MOTU delimitation at a constant 99% similarity threshold that produces the per-location richness and read-count table this skill tests)
- **custom R scripts (metabarpark)** (Publicly available R scripts implementing the GLM/Tukey analysis of the MOTU table) — http://github.com/metabarpark

## Examples

```
fit <- glm2(richness ~ factor(location), family = poisson); summary(multcomp::glht(fit, linfct = mcp(location = 'Tukey')))
```

## Evaluation signals

- Tukey contrast table reproduces the reported P-values: Bahamas vs Jamaica P=0.039, Bahamas vs Belize P=0.095, Chesterfield vs Noumea P=0.013, Bahamas vs other Caribbean P<0.001, Turks & Caicos vs Jamaica P=0.033 and vs Belize P=0.047.
- New Caledonia read-abundance pairwise contrasts all return P<1e-10.
- The abundance response is square-root transformed before fitting, while MOTU richness enters the Poisson GLM as raw counts.
- Model objects are glm2 Poisson fits with location as a categorical factor (categorical levels, not a continuous or ordinal covariate).
- glm_tukey_results.csv exists and contains both the model summaries and the Tukey contrast tables for the two response variables.

## Limitations

- Primer bias and species-specific shedding rates limit quantitative interpretation of read counts, so abundance contrasts are only a potentially relative abundance measure.
- Single-read MOTUs were removed as a conservative contamination/false-positive control, so per-location richness values depend on that filtering decision.
- No experiment verifies the relationship between eDNA read abundance and community biomass for elasmobranchs.
- Taxonomic resolution of the 127 bp COI fragment cannot separate closely related Carcharhinus, Rhizoprionodon and Negaprion species; some MOTUs are 100% identical to multiple BOLD species.

## Evidence

- [discussion] Generalized linear model analysis of MOTU richness and read abundances by location: "A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location"
- [intro] Study tests whether species diversity and eDNA read abundance reflect anthropogenic impact in two tropical marine systems: "and specifically examine whether patterns of species diversity and eDNA read abundance, reflect the known degree of anthropogenic impact in two independent tropical marine systems"
- [discussion] Remote/protected locations show the highest species richness and read abundance, giving the expected direction of GLM contrasts: "Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance among our samples"
- [discussion] Single-read MOTUs removed as conservative contamination/false-positive control before richness counting: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
