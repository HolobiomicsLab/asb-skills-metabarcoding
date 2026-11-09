---
name: read-abundance-variance-stabilization
description: Use when comparing eDNA metabarcoding read counts or MOTU richness across
  discrete sampling locations and you need a Poisson GLM framework with post-hoc Tukey
  contrasts (multcomp) to test pairwise differences.
when_to_use_negative:
- Input is still raw sequencing output (FASTQ/aligned reads) rather than a collapsed
  per-location MOTU count table — upstream OBITools/sumaclust processing is required
  first
- The response variable is taxonomic assignment or diversity estimates unrelated to
  counts; the Poisson GLM and square-root transform target MOTU richness and read
  abundance specifically
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: R (glm2, multcomp)
  role: glm2 fits the Poisson GLM with location as categorical factor; multcomp performs
    post-hoc Tukey pairwise contrasts
  repo: http://github.com/metabarpark
- name: sumaclust
  role: 'Upstream: delimits MOTUs at a 99% similarity threshold, producing the MOTU
    table from which per-location richness and read abundance are tallied'
- name: OBITools
  role: 'Upstream: bioinformatic suite (illuminapairedend, ngsfilter, obigrep, obiuniq,
    vsearch/uchime-denovo, ecotag) that produces the filtered, dereplicated reads
    clustered by sumaclust'
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:fe81e22016d6c1df2e047fc06df9cabf3d12e94ecb516552cd8be1d6a8887639
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - R (glm2, multcomp)
  - sumaclust
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# read-abundance-variance-stabilization

## Summary

Variance-stabilizing preprocessing and count-modeling step from Bakker et al. 2017 in which eDNA sequence read abundances are square-root transformed before being tested against location with a Poisson generalized linear model, alongside untransformed MOTU richness.

## When to use

Use when comparing eDNA metabarcoding read counts or MOTU richness across discrete sampling locations and you need a Poisson GLM framework with post-hoc Tukey contrasts (multcomp) to test pairwise differences.

## When NOT to use

- Input is still raw sequencing output (FASTQ/aligned reads) rather than a collapsed per-location MOTU count table — upstream OBITools/sumaclust processing is required first
- The response variable is taxonomic assignment or diversity estimates unrelated to counts; the Poisson GLM and square-root transform target MOTU richness and read abundance specifically

## Inputs

- Per-location MOTU richness and read abundance table (derived from the sumaclust-delimited MOTU table of the OBITools pipeline)
- Location grouping factor for Caribbean (Bahamas, Belize, Jamaica, Turks & Caicos) and New Caledonia (Chesterfield, Noumea) samples

## Outputs

- glm_tukey_results.csv with model summaries and Tukey contrast tables
- Pairwise P-values for MOTU richness and square-root-transformed read abundance contrasts among locations

## How to apply

Load the per-location table of MOTU richness and read abundance derived from the metabarcoding MOTU table (OBITools/sumaclust 99% similarity clustering pipeline). Square-root transform the sequence read abundances, as specified for the abundance response variable, while leaving MOTU richness untransformed because it is already count-like. Fit a Poisson generalized linear model using glm2 with location entered as a categorical factor, once per response variable. Apply post-hoc Tukey pairwise contrasts with the multcomp package and extract P-values (e.g. Bahamas vs Jamaica P=0.039, Bahamas vs Belize P=0.095, Chesterfield vs Noumea P=0.013). Rationale: square-root transformation stabilizes the extreme variance typical of eDNA read counts, which otherwise violate Poisson assumptions, and the categorical GLM directly tests location effects that reflect anthropogenic pressure differences.

## Related tools

- **R (glm2, multcomp)** (glm2 fits the Poisson GLM with location as categorical factor; multcomp performs post-hoc Tukey pairwise contrasts) — http://github.com/metabarpark
- **sumaclust** (Upstream: delimits MOTUs at a 99% similarity threshold, producing the MOTU table from which per-location richness and read abundance are tallied)
- **OBITools** (Upstream: bioinformatic suite (illuminapairedend, ngsfilter, obigrep, obiuniq, vsearch/uchime-denovo, ecotag) that produces the filtered, dereplicated reads clustered by sumaclust)

## Examples

```
m <- glm2(sqrt_reads ~ location, family=poisson, data=tab); summary(glht(m, linfct=mcp(location="Tukey")))
```

## Evaluation signals

- Tukey contrast P-values reproduce the reported values: Bahamas vs Jamaica P=0.039, Bahamas vs Belize P=0.095, Chesterfield vs Noumea P=0.013, Bahamas vs other Caribbean P<0.001, Turks & Caicos vs Jamaica P=0.033 and vs Belize P=0.047
- New Caledonia read-abundance comparisons yield P<1e-10 under the Poisson GLM
- The abundance response is square-root transformed before modeling; MOTU richness is modeled as raw counts
- Model summary and Tukey contrast tables are written to glm_tukey_results.csv with pairwise contrasts matching the reported directionality (protected/remote sites highest richness and abundance)

## Limitations

- No experiments verify the relationship between eDNA read abundance and community biomass for elasmobranchs, so abundance differences are relative, not biomass estimates
- Shedding rates and primer efficiency may differ among species templates (primer bias), limiting quantitative interpretation of read counts
- Single-read MOTUs were conservatively removed as potential contamination/sequencing-error false positives, which affects the richness counts fed into the GLM

## Evidence

- [discussion] GLM on square-root transformed read abundance: "A generalized linear model approach was used for testing differences in MOTU richness and read abundances (square-root transformed) as a function of location"
- [abstract] diversity and abundance reflect anthropogenic impact: "whose geographical patterns of diversity and read abundance coincide with geographical differences in levels of anthropogenic pressure and conservation effort"
- [discussion] MOTUs delimited by sumaclust at 99%: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [readme] custom R scripts available at metabarpark: "We are working on DNA-metabarcoding of marine benthic communities."
