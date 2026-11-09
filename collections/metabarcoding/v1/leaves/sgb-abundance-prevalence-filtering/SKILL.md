---
name: sgb-abundance-prevalence-filtering
description: Use when a MetaPhlAn 4 SGB-level profile (e.g., 181 mouse gut metagenomes)
  must be reduced to prevalent taxa before per-taxon models such as diet biomarker
  linear mixed models.
when_to_use_negative:
- The input is already a filtered feature/abundance table (re-filtering would discard
  taxa needed for the final biomarker set)
- The goal is strain-level or functional profiling (e.g., StrainPhlAn population genomics
  or HUMAnN pathways) rather than taxon-level abundance filtering
- Environmental, nonhost-associated, or under-studied microbial communities where
  most sequence content falls outside current uSGBs and an abundance cutoff would
  discard the signal of interest
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0219
- http://edamontology.org/topic_0610
tools:
- name: MetaPhlAn 4
  role: Produces the SGB-level relative abundance profiles (26,970 SGBs, including
    4,992 taxonomically unidentified uSGBs) that are subject to the abundance-prevalence
    filter
  repo: https://github.com/biobakery/MetaPhlAn
- name: biobakery
  role: BioBakery platform tutorials/wiki context for MetaPhlAn profiling workflows
    that generate the input profiles
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:f64401e1a332c38c7768418df82387644ffc8bfb6cc3a1308433387710c31807
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn 4
  - biobakery
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0219
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sgb-abundance-prevalence-filtering

## Summary

Filters a MetaPhlAn 4 SGB-level relative abundance table to taxa with mean relative abundance greater than 1% across the cohort prior to downstream association testing, reducing sparse taxa that cannot yield reliable biomarker signals.

## When to use

Use when a MetaPhlAn 4 SGB-level profile (e.g., 181 mouse gut metagenomes) must be reduced to prevalent taxa before per-taxon models such as diet biomarker linear mixed models.

## When NOT to use

- The input is already a filtered feature/abundance table (re-filtering would discard taxa needed for the final biomarker set)
- The goal is strain-level or functional profiling (e.g., StrainPhlAn population genomics or HUMAnN pathways) rather than taxon-level abundance filtering
- Environmental, nonhost-associated, or under-studied microbial communities where most sequence content falls outside current uSGBs and an abundance cutoff would discard the signal of interest

## Inputs

- MetaPhlAn 4 SGB-level relative abundance profile (taxa-by-sample matrix) for a metagenome cohort (e.g., 181 mouse gut metagenomes)
- Host/sample metadata table (diet group, sex, age, genetic background, vendor)

## Outputs

- Filtered SGB abundance table restricted to taxa with mean relative abundance > 1% across the cohort
- Pre-processed CLR-transformed abundance matrix (with pseudocount applied to zeros) ready for per-SGB modeling

## How to apply

Load the MetaPhlAn 4 SGB relative abundance table (species-level genome bins, including uSGBs) as a taxa-by-sample matrix. Apply a centered log-ratio (CLR) transform first, adding a pseudocount to zero abundances prior to the log transform. Then compute each SGB's mean relative abundance across the full cohort and retain only taxa exceeding 1% mean abundance. This abundance-prevalence cutoff precedes per-SGB linear mixed modeling and Benjamini-Hochberg FDR correction (threshold FDR < 0.2 in the mouse high-fat diet biomarker workflow), because low-abundance taxa contribute unstable compositional signals. The retained table feeds downstream effect-size estimation for biomarkers such as Lachnospiraceae bacterium 28_4 SGB7272, Lactobacillus johnsonii SGB7041, and Faecalibaculum rodentium SGB4047.

## Related tools

- **MetaPhlAn 4** (Produces the SGB-level relative abundance profiles (26,970 SGBs, including 4,992 taxonomically unidentified uSGBs) that are subject to the abundance-prevalence filter) — https://github.com/biobakery/MetaPhlAn
- **biobakery** (BioBakery platform tutorials/wiki context for MetaPhlAn profiling workflows that generate the input profiles) — https://github.com/biobakery/biobakery

## Examples

```
import pandas as pd; df = pd.read_csv('metaphlan_sgb_relab.tsv', sep='\t', index_col=0); filtered = df.loc[df.mean(axis=1) > 0.01]
```

## Evaluation signals

- The output table contains only SGBs whose mean relative abundance across the cohort exceeds 1%
- In the mouse diet biomarker validation, filtering yields 18 SGB biomarkers at FDR < 0.2, with 13 of 18 (~72%) being uSGBs
- Anchor taxa are retained in the output: SGB7272 (Lachnospiraceae bacterium 28_4), SGB7041 (Lactobacillus johnsonii), and SGB4047 (Faecalibaculum rodentium)
- All abundance values were CLR-transformed with a pseudocount added to zeros before the log transform (no raw-zero log values present)
- The filtered table schema matches diet_biomarkers_sgb.tsv expectations: SGB IDs with effect sizes, p-values, and FDR values

## Limitations

- An abundance-prevalence cutoff discards low-prevalence taxa, trading sensitivity for specificity — the article flags this 'tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa' as an unaddressed downstream statistical challenge
- Environmental, nonhost-associated, and under-studied communities remain highly enriched for sequences not captured by current uSGBs, so a 1% filter may disproportionately exclude genuinely present uncharacterized taxa in such settings

## Evidence

- [other] Filter the SGB table to taxa with mean relative abundance greater than 1% across the cohort.: [evidence span withheld — anchor status: not_found]
- [results] SGB biomarkers via FDR 0.2: [evidence span withheld — anchor status: not_found]
- [abstract] uSGB biomarker finding: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [readme] MetaPhlAn scope: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
