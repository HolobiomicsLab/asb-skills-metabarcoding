---
name: diet-biomarker-identification-from-taxonomic-profiles
description: Use when metagenomic taxonomic profiles (e.g. MetaPhlAn 4 SGB-level relative
  abundances) must be tested for differential abundance between two host conditions,
  such as high-fat vs normal-chow diet, while controlling for host covariates.
when_to_use_negative:
- Input is raw sequencing reads rather than a taxonomic abundance table — profile
  first with MetaPhlAn 4 before biomarker testing
- The outcome of interest is functional potential rather than taxonomic abundance
  (use HUMAnN-style functional profiling instead)
- Environments dominated by sequences not captured by current uSGBs (e.g. environmental
  or nonhost-associated communities), where uSGB biomarker fractions will be unreliable
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaPhlAn 4
  role: Produces the SGB-level relative abundance profiles (including uSGBs) that
    serve as the input feature table for the diet biomarker association models
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: General species-level taxonomic profiling tool from which MetaPhlAn 4 extends
    marker gene coverage to uncharacterized species
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:4873ced1072bf5a5cfcdb9f4094e1f017f74f88efc1799a222c28ee2fc37e338
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# diet-biomarker-identification-from-taxonomic-profiles

## Summary

Identify taxonomic biomarkers (SGBs) associated with a binary diet condition by fitting per-taxon linear mixed models on CLR-transformed MetaPhlAn 4 relative abundances, with covariate control and Benjamini-Hochberg FDR correction, and quantify the fraction of biomarkers that are uncharacterized species (uSGBs).

## When to use

Use when metagenomic taxonomic profiles (e.g. MetaPhlAn 4 SGB-level relative abundances) must be tested for differential abundance between two host conditions, such as high-fat vs normal-chow diet, while controlling for host covariates.

## When NOT to use

- Input is raw sequencing reads rather than a taxonomic abundance table — profile first with MetaPhlAn 4 before biomarker testing
- The outcome of interest is functional potential rather than taxonomic abundance (use HUMAnN-style functional profiling instead)
- Environments dominated by sequences not captured by current uSGBs (e.g. environmental or nonhost-associated communities), where uSGB biomarker fractions will be unreliable

## Inputs

- MetaPhlAn 4 SGB-level relative abundance profile table (e.g. for 181 mouse gut metagenomes)
- Host/sample metadata table with diet group, sex, age, genetic background and vendor
- SGB-to-taxonomy mapping to classify biomarkers as characterized species vs uSGBs

## Outputs

- diet_biomarkers_sgb.tsv listing surviving SGB biomarkers with effect sizes, p-values and FDR values
- Summary statistic of the fraction of diet biomarkers classified as uncharacterized species (uSGBs)

## How to apply

Load MetaPhlAn 4 SGB-level relative abundance profiles together with host metadata (diet group, sex, age, genetic background, vendor) for the cohort (e.g. 181 mouse gut metagenomes). Apply a centered log-ratio (CLR) transform to the compositional SGB table, adding a pseudocount to zero abundances before the log transform, and filter to SGBs with mean relative abundance greater than 1% across the cohort. Fit a random-intercept linear mixed model per SGB (lme4 or statsmodels MixedLM) with diet (HF vs chow) as the fixed effect, sex, age, genetic background and vendor as covariates, and a random intercept to absorb cohort non-independence. Correct per-SGB diet association p-values with Benjamini-Hochberg FDR and retain SGBs with FDR < 0.2, compiling effect sizes, p-values and FDRs into a table; report the fraction of biomarkers that are uSGBs (taxonomically unidentified species-level genome bins in the MetaPhlAn 4 database).

## Related tools

- **MetaPhlAn 4** (Produces the SGB-level relative abundance profiles (including uSGBs) that serve as the input feature table for the diet biomarker association models) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (General species-level taxonomic profiling tool from which MetaPhlAn 4 extends marker gene coverage to uncharacterized species) — https://github.com/biobakery/MetaPhlAn

## Examples

```
conda install -c bioconda metaphlan && metaphlan reads.fastq --input_type fastq --bowtie2db metaphlan_db --output profile.tsv
```

## Evaluation signals

- Output table contains the expected 18 SGB biomarkers called at FDR < 0.2 and mean relative abundance > 1%
- Anchor SGBs Lachnospiraceae bacterium 28_4 SGB7272, Lactobacillus johnsonii SGB7041 and Faecalibaculum rodentium SGB4047 are present in the biomarker table
- Majority (~72%) of biomarkers are classified as uSGBs (13 of 18), consistent with the reported finding that they are over-abundant on a hyper-caloric diet
- All retained SGBs passed the mean relative abundance > 1% filter and have Benjamini-Hochberg FDR < 0.2 with reported effect sizes
- CLR transformation used a pseudocount added to zero abundances prior to the log transform, and models included sex, age, genetic background and vendor as covariates with a random intercept

## Limitations

- Biomarker calls at FDR < 0.2 are a lenient threshold; a large fraction (72%) of diet-associated taxa may be taxonomically unidentified uSGBs, limiting interpretability at the species level
- Quality control on rare taxa involves a tradeoff between sensitivity and specificity that remains an unaddressed downstream statistical challenge
- MetaPhlAn 4 does not extensively incorporate viral or eukaryotic microbial sequences, so viral/eukaryotic diet biomarkers are out of scope

## Evidence

- [abstract] Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [abstract] uSGB count: "we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [discussion] biomarker resolution: "This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa"
- [discussion] rare taxa QC tradeoff: "the tradeoff between sensitivity and specificity when applying quality control measures to identify real but"
- [methods] viral module scope: "If you use the MetaPhlAn viral module, please cite the MetaPhlAn paper and the MetaPhlan viral module paper"
- [readme] MetaPhlAn install: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [readme] MetaPhlAn scope: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
