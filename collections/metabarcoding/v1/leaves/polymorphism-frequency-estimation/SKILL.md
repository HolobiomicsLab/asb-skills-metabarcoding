---
name: polymorphism-frequency-estimation
description: Use when full-length 16S amplicon reads (e.g., PacBio CCS from stool
  samples) assigned to a 99% identity OTU must be characterized by per-position substitution
  frequencies to detect and quantify intragenomic 16S copy variants in vivo.
when_to_use_negative:
- When the target polymorphism is an insertion or deletion — PacBio CCS deletion errors
  coincident with homopolymer runs make indel calls unreliable, so only substitutions
  are trustworthy
- When reads are short-read amplicon data of 16S variable regions (e.g., V4), which
  cannot achieve the taxonomic resolution of the full ~1500 bp gene
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3174
tools:
- name: PacBio
  role: Long-read sequencing platform producing full-length ~1500 bp 16S CCS amplicon
    reads sufficiently accurate to resolve single-nucleotide substitutions between
    intragenomic 16S copies
- name: Cross_match
  role: Aligner used to align B. vulgatus OTU amplicons to the reference 16S gene
    for per-position substitution tabulation
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the analysis code into figure-specific
    pipelines (e.g., pipeline_pb_strains for Figure 4)
  repo: https://github.com/cgat-developers/ruffus
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:491722a113c81b40c1c311af9e1ba5af18933192116fe26a23ae7ddd2eca945e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - Cross_match
  - PacBio
  - ruffus
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# polymorphism-frequency-estimation

## Summary

Estimates the frequency of intragenomic 16S gene copy polymorphisms within a taxon by tabulating per-position nucleotide substitution frequencies across full-length PacBio 16S amplicons aligned to a reference gene, enabling strain-level inference (e.g., B. vulgatus V5 polymorphism frequencies in stool samples).

## When to use

Use when full-length 16S amplicon reads (e.g., PacBio CCS from stool samples) assigned to a 99% identity OTU must be characterized by per-position substitution frequencies to detect and quantify intragenomic 16S copy variants in vivo.

## When NOT to use

- When the target polymorphism is an insertion or deletion — PacBio CCS deletion errors coincident with homopolymer runs make indel calls unreliable, so only substitutions are trustworthy
- When reads are short-read amplicon data of 16S variable regions (e.g., V4), which cannot achieve the taxonomic resolution of the full ~1500 bp gene

## Inputs

- Full-length PacBio CCS 16S amplicon reads (e.g., from github.com/TheJacksonLaboratory/weinstock_full_length_16s), assigned to a 99% identity OTU
- Single-taxon reference 16S gene sequence (e.g., B. vulgatus)
- RefSeq 16S sequences for comparison strains (e.g., B. vulgatus ATCC 8482, mpk40)

## Outputs

- Per-position substitution profiles by sample and reference strain (substitution_profiles.csv)
- Polymorphism frequency estimates per sample (e.g., V5 polymorphism at 84% Scott, 69% IronHorse)

## How to apply

Retrieve full-length PacBio 16S amplicon reads and select those belonging to the taxon of interest (e.g., the B. vulgatus 99% identity OTU from the Scott and IronHorse samples). Align the amplicons to a single reference 16S gene sequence (plus RefSeq 16S sequences for comparison strains such as ATCC 8482 and mpk40) using Cross_match. Tabulate nucleotide substitutions at each aligned position, deliberately ignoring insertions and deletions because systematic PacBio CCS deletion errors coincident with homopolymer runs make indel calls unreliable. Compute the fraction of reads carrying each polymorphism (e.g., the V5 polymorphism) per sample and compare the observed frequencies to those predicted from the reference strain's 16S gene copy composition to infer which strain(s) are present.

## Related tools

- **PacBio** (Long-read sequencing platform producing full-length ~1500 bp 16S CCS amplicon reads sufficiently accurate to resolve single-nucleotide substitutions between intragenomic 16S copies)
- **Cross_match** (Aligner used to align B. vulgatus OTU amplicons to the reference 16S gene for per-position substitution tabulation)
- **ruffus** (CGAT ruffus pipeline framework used to organize the analysis code into figure-specific pipelines (e.g., pipeline_pb_strains for Figure 4)) — https://github.com/cgat-developers/ruffus

## Examples

```
python pipeline_pb_strains.py --otu b_vulgatus --samples Scott,IronHorse --references ATCC_8482,mpk40 --out substitution_profiles.csv
```

## Evaluation signals

- Computed V5 polymorphism frequency matches reported values: 84% in the Scott sample and 69% in the IronHorse sample
- Per-position substitution profiles reproduce the patterns shown in Fig. 3c and 3d of the paper
- Observed polymorphism frequencies are consistent with the copy-number fractions predicted from the reference strain (e.g., 6/7 ≈ 84% and 5/7 ≈ 69% of 16S copies)
- Insertions and deletions are excluded from the substitution tabulation, with only substitution calls retained
- Output file substitution_profiles.csv contains per-position profiles organized by sample and reference strain

## Limitations

- Systematic PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences, forcing the analysis to ignore insertions/deletions
- Systematic errors may occur at a given frequency and hence may not be improved by greater sequencing effort
- The approach of ignoring insertions/deletions is imperfect, as demonstrated by a single deletion in one of the seven E. coli strain K-12 substr. MG1655 16S genes

## Evidence

- [abstract] full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene: "full-length sequencing platforms are sufficiently accurate to resolve subtle nucleotide substitutions (but not insertions/deletions) that exist between intragenomic copies of the 16S gene"
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
- [discussion] full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [readme] Code is organised into a set of ruffus pipelines pertaining to different figures: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures."
