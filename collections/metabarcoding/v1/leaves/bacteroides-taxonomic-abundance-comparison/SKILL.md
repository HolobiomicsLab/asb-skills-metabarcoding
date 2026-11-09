---
name: bacteroides-taxonomic-abundance-comparison
description: Use when 16S amplicon reads (full-length V1–V9 or partial V1–V3) from
  stool samples must be resolved to Bacteroides species-level relative abundances
  and cross-checked against mWGS quantification.
when_to_use_negative:
- Short-read sequencing of a single 16S variable region (e.g., V4) is the only data
  available and species-level resolution is required — the paper shows none of the
  partial regions recaptured full-gene diversity
- The goal is strain-level discrimination via intragenomic 16S copy variants — that
  requires the paper's SNP-phasing substitution-profile approach, not 99% OTU clustering
- Input is already a taxonomic abundance table rather than raw/dereplicated amplicon
  reads
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: USEARCH
  role: Dereplication, 99%-identity OTU clustering of 16S reads, and alignment of
    Bacteroides OTU representative sequences to the RTG reference 16S database for
    species-level taxonomy assignment
- name: PacBio
  role: Long-read platform producing full-length (V1–V9) 16S amplicon reads that enable
    species-level Bacteroides resolution
- name: cgat-developers/ruffus
  role: CGAT ruffus pipeline framework used to run the figure-specific analysis pipelines
    from the weinstock_full_length_16s repository
  repo: https://github.com/cgat-developers/ruffus
- name: TheJacksonLaboratory/weinstock_full_length_16s
  role: Depository of preprocessed stool amplicon read sets and the analysis code
    underlying Figures 1–3
  repo: https://github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:801911cc2ebc0111edfec603ba07cc61152c75bab1192354274cdc08f58c9d2a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - TheJacksonLaboratory/weinstock_full_length_16s
  - USEARCH
  - cgat-developers/ruffus
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# bacteroides-taxonomic-abundance-comparison

## Summary

Reproduce genus- and species-level Bacteroides quantification from 99%-identity OTU clustering of full-length (V1–V9, PacBio) and partial (V1–V3, MiSeq) 16S amplicons, and validate the profiles against matched metagenomic whole-genome-shotgun (mWGS) abundances in four human stool samples.

## When to use

Use when 16S amplicon reads (full-length V1–V9 or partial V1–V3) from stool samples must be resolved to Bacteroides species-level relative abundances and cross-checked against mWGS quantification.

## When NOT to use

- Short-read sequencing of a single 16S variable region (e.g., V4) is the only data available and species-level resolution is required — the paper shows none of the partial regions recaptured full-gene diversity
- The goal is strain-level discrimination via intragenomic 16S copy variants — that requires the paper's SNP-phasing substitution-profile approach, not 99% OTU clustering
- Input is already a taxonomic abundance table rather than raw/dereplicated amplicon reads

## Inputs

- Preprocessed PacBio full-length (V1–V9) 16S amplicon reads from human stool samples (from github.com/TheJacksonLaboratory/weinstock_full_length_16s)
- Preprocessed MiSeq partial (V1–V3) 16S amplicon reads from the same stool samples
- mWGS (metagenomic whole-genome-shotgun) genus- and species-level abundance profiles for the matched samples
- RTG reference 16S database for species-level taxonomy assignment

## Outputs

- 99%-identity OTU tables per sequencing platform/region (V1–V9 and V1–V3)
- Bacteroides species-level taxonomy assignments for OTU representative sequences
- Genus- and species-level Bacteroides relative abundance profiles per individual for each 16S platform
- Comparison of 16S-derived vs mWGS abundances reproducing Fig. 3a–b (high/low-Bacteroides grouping concordance)

## How to apply

Load the preprocessed stool amplicon read sets (PacBio V1–V9 full-length and MiSeq V1–V3) from github.com/TheJacksonLaboratory/weinstock_full_length_16s, where the paper's Fig. 1–3 pipelines are organised as CGAT ruffus pipelines. Dereplicate and cluster each platform's reads into OTUs at 99% sequence identity using USEARCH — the paper found that full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance. Align Bacteroides OTU representative sequences to the RTG reference 16S database with USEARCH to assign species-level taxonomy. Compute genus- and species-level relative abundance profiles per individual and compare the 16S-derived abundances with matched mWGS quantification, reproducing the Fig. 3a–b comparisons. Finally, verify that individuals group concordantly as high- vs low-Bacteroides between full-length 16S and mWGS, and confirm the known failure mode: V1–V3 MiSeq data miss the abundant Bacteroides intestinalis in one sample.

## Related tools

- **USEARCH** (Dereplication, 99%-identity OTU clustering of 16S reads, and alignment of Bacteroides OTU representative sequences to the RTG reference 16S database for species-level taxonomy assignment)
- **PacBio** (Long-read platform producing full-length (V1–V9) 16S amplicon reads that enable species-level Bacteroides resolution)
- **cgat-developers/ruffus** (CGAT ruffus pipeline framework used to run the figure-specific analysis pipelines from the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus
- **TheJacksonLaboratory/weinstock_full_length_16s** (Depository of preprocessed stool amplicon read sets and the analysis code underlying Figures 1–3) — https://github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Examples

```
python pipeline_bacteroides.py --reads V1V9.fastq V1V3.fastq --usearch usearch --id 0.99 --refdb RTG_16S --mwgs mwgs_abundances.tsv
```

## Evaluation signals

- Full-length (V1–V9) 99% OTU-based Bacteroides species relative abundances provide reasonable estimates of the mWGS-measured species abundances, per the paper's reported finding
- Individuals group concordantly as high- vs low-Bacteroides between full-length 16S profiles and mWGS quantification
- Bacteroides intestinalis, abundant in mWGS and full-length 16S data, is absent from the V1–V3 MiSeq species profiles for one sample
- OTU clustering was performed at 99% sequence identity (not the conventional 97%), consistent with the paper's workflow
- Bacteroides OTU representative sequences were assigned species-level taxonomy by USEARCH alignment against the RTG reference 16S database

## Limitations

- V1–V3 (MiSeq) sequencing failed to detect the abundant Bacteroides intestinalis in one sample, so partial-region profiles are incomplete at species level
- PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences; the analysis compensates by focusing on substitutions and ignoring insertions/deletions
- The 99% identity threshold's ability to separate intragenomic vs. inter-genomic 16S variation was not determined for all bacterial species
- None of the variable regions covered by partial 16S sequencing recaptured the diversity represented by the full ~1500 bp gene

## Evidence

- [discussion] full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [discussion] none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [readme] Code is organised into a set of ruffus pipelines pertaining to different figures. Each pipeline is designed to be run using a CGAT ruffus framework.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining to different figures. Each pipeline is designed to be run using a
[CGAT ruffus"
- [readme] The code underlying Figures 1-3 is written in Python 2.7.: "The code
underlying Figures 1-3 is written in Python 2.7."
- [discussion] we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions: "we chose to overcome such platform-specific errors by focusing on substitutions and ignoring the contribution of insertions and deletions to intragenomic 16S gene copy polymorphisms"
