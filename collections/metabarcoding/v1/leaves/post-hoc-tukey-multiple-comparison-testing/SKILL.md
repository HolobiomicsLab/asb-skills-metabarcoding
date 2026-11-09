---
name: post-hoc-tukey-multiple-comparison-testing
description: Use when you have per-location count data (e.g. MOTU richness or eDNA
  read counts from metabarcoding) and need statistically justified pairwise contrasts
  among a categorical grouping factor after a GLM.
when_to_use_negative:
- Input is still raw FASTQ or an unfiltered MOTU table — the metabarcoding pipeline
  (illumincapairedend, ngsfilter, obigrep 120–135 bp, uchime-denovo, sumaclust 99%)
  must be run and single-read MOTUs removed first
- The response variable is continuous and non-count-like, making a Poisson GLM inappropriate
- Only two groups are compared with no multiplicity concern — a simple pairwise test
  suffices and Tukey correction adds nothing
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: glm2
  role: Fits the Poisson generalized linear model with location as a categorical factor
    for MOTU richness and square-root-transformed read abundance
- name: multcomp
  role: R package used to run the post-hoc Tukey pairwise contrasts among sampling
    locations
- name: R
  role: Analysis environment; the paper's custom R scripts are publicly available
    from github.com/metabarpark
  repo: github.com/metabarpark
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:06c5f1696a961ebe2654404c9f47defc4537a0e581409fd2a2149930268bd8e3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/github.com/metabarpark
  tools:
  - R
  - glm2
  - multcomp
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# post-hoc-tukey-multiple-comparison-testing

## Summary

Fits Poisson generalized linear models with location as a categorical factor on shark eDNA MOTU richness and square-root-transformed read abundances, then extracts all pairwise location contrasts with Tukey-adjusted post-hoc comparisons. Used to test whether elasmobranch diversity and read abundance differ significantly among Caribbean and New Caledonia sampling sites.

## When to use

Use when you have per-location count data (e.g. MOTU richness or eDNA read counts from metabarcoding) and need statistically justified pairwise contrasts among a categorical grouping factor after a GLM.

## When NOT to use

- Input is still raw FASTQ or an unfiltered MOTU table — the metabarcoding pipeline (illumincapairedend, ngsfilter, obigrep 120–135 bp, uchime-denovo, sumaclust 99%) must be run and single-read MOTUs removed first
- The response variable is continuous and non-count-like, making a Poisson GLM inappropriate
- Only two groups are compared with no multiplicity concern — a simple pairwise test suffices and Tukey correction adds nothing

## Inputs

- Per-location table of shark eDNA MOTU richness (integer counts) derived from the sumaclust MOTU table
- Per-location eDNA sequence read abundance values (to be square-root transformed)
- Location labels as a categorical factor (Caribbean: Bahamas, Belize, Jamaica, Turks & Caicos; New Caledonia: Chesterfield, Noumea)

## Outputs

- Poisson GLM model summaries for MOTU richness and square-root-transformed read abundance
- Tukey-adjusted pairwise location contrast table with P-values
- glm_tukey_results.csv containing model summaries and Tukey contrast tables

## How to apply

Load the per-location MOTU richness and read abundance table derived from the OBITools/sumaclust MOTU table (99% similarity clustering, single-read MOTUs removed). Square-root transform the sequence read abundance response, as specified for the abundance variable, while keeping MOTU richness as raw counts. Fit a Poisson GLM with glm2, entering location as a categorical factor, separately for each response variable. Apply post-hoc Tukey contrasts with the multcomp package (glht with mcp(location='Tukey')) and extract the adjusted pairwise P-values. Write the model summaries and Tukey contrast tables to glm_tukey_results.csv; correct reproduction is judged by matching the reported contrasts (e.g. Bahamas vs Jamaica P=0.039, Chesterfield vs Noumea P=0.013).

## Related tools

- **glm2** (Fits the Poisson generalized linear model with location as a categorical factor for MOTU richness and square-root-transformed read abundance)
- **multcomp** (R package used to run the post-hoc Tukey pairwise contrasts among sampling locations)
- **R** (Analysis environment; the paper's custom R scripts are publicly available from github.com/metabarpark) — github.com/metabarpark

## Examples

```
fit <- glm2(richness ~ location, family = poisson, data = motu_tbl); summary(glht(fit, linfct = mcp(location = "Tukey")))
```

## Evaluation signals

- Bahamas vs Jamaica MOTU richness contrast reproduces P = 0.039 and Bahamas vs Belize reproduces P = 0.095
- Chesterfield vs Noumea contrast reproduces P = 0.013 and Bahamas vs other Caribbean sites P < 0.001
- Turks & Caicos vs Jamaica P = 0.033 and vs Belize P = 0.047; New Caledonia abundance contrasts P < 1e-10
- Read abundance values were square-root transformed before model fitting (not raw counts)
- Output file glm_tukey_results.csv exists and contains both model summaries and the Tukey contrast tables

## Limitations

- All single-read MOTUs were removed as a conservative contamination/false-positive control, which affects richness counts used as the response
- No experiments have verified the relationship between eDNA read abundance and community biomass for elasmobranchs, so abundance-based contrasts have ambiguous quantitative interpretation
- Primer bias and differing shedding rates among species limit quantitative interpretation of read counts
- Taxonomic resolution of the 127 bp COI fragment cannot separate closely related species (e.g. Carcharhinus, Rhizoprionodon, Negaprion), which can affect MOTU-level richness definitions

## Evidence

- [discussion] A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location: "A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location"
- [discussion] All custom R scripts are publicly available from http://github.com/metabarpark.: "All custom R scripts are publicly available from http://github.com/metabarpark."
- [other] Apply post-hoc Tukey comparisons using the multcomp package to obtain pairwise contrasts among sampling locations for each response variable.: [evidence span withheld — anchor status: not_found]
- [discussion] MOTU richness and read abundance patterns follow the level of anthropogenic impact in each location: [evidence span withheld — anchor status: not_found]
