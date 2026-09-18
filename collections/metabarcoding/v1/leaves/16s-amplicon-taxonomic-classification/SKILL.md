---
name: 16s-amplicon-taxonomic-classification
description: Use when assessing whether a 16S variable region (e.g. V4) or full-length
  V1-V9 amplicon can achieve confident species-level taxonomy against a reference
  database such as Greengenes.
when_to_use_negative:
- Input is already a classified OTU/feature table or taxonomy assignment — re-classifying
  with classify.seqs adds nothing
- The goal is strain- or SNP-level discrimination of intragenomic 16S copies; the
  paper addresses that via full-length sequencing and 99% identity clustering, not
  Wang-classifier species assignment
- Amplicons come from a mock/in-vivo community without known source species, so the
  species-level failure rate against ground truth cannot be computed
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: mothur
  role: Runs classify.seqs with the Wang Bayesian classifier to assign each amplicon
    a taxonomic lineage against the Greengenes reference database at swept confidence
    cutoffs
- name: ruffus
  role: CGAT ruffus pipeline framework used to orchestrate the per-figure analysis
    pipelines (Figures 1-3 written in Python 2.7), including this classification step
  repo: github.com/cgat-developers/ruffus
- name: weinstock_full_length_16s
  role: Repository containing the code used to perform this in-silico 16S classification
    analysis reported in Johnson et al.
  repo: github.com/TheJacksonLaboratory/weinstock_full_length_16s
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:bc5c817eb742def151f87b000aa1d68692c305ceddbd8fdc70dd5b6ba7af682e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - mothur
  - ruffus
  - weinstock_full_length_16s
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# 16s-amplicon-taxonomic-classification

## Summary

Classifies 16S rRNA amplicon sequences (in-silico or sequenced) against a Greengenes reference taxonomy using the mothur Wang Bayesian classifier, sweeping confidence cutoffs to quantify species-level classification failure per variable region. This skill grounds the paper's finding that the V4 sub-region fails to achieve confident species-level assignment for 56% of amplicons, while full-length V1-V9 nearly recovers it.

## When to use

Use when assessing whether a 16S variable region (e.g. V4) or full-length V1-V9 amplicon can achieve confident species-level taxonomy against a reference database such as Greengenes.

## When NOT to use

- Input is already a classified OTU/feature table or taxonomy assignment — re-classifying with classify.seqs adds nothing
- The goal is strain- or SNP-level discrimination of intragenomic 16S copies; the paper addresses that via full-length sequencing and 99% identity clustering, not Wang-classifier species assignment
- Amplicons come from a mock/in-vivo community without known source species, so the species-level failure rate against ground truth cannot be computed

## Inputs

- In-silico 16S amplicon FASTA sequences per sub-region (V1-V9 sub-regions and full-length) derived from Greengenes reference genomes
- Greengenes reference alignment file
- Greengenes reference taxonomy file
- Mapping of each amplicon to its known source reference genome/species (from the task_001 primer extraction step)

## Outputs

- mothur classify.seqs taxonomy assignment files at each confidence cutoff (30–98)
- Per-region species-level failure rate table written to fig1b_species_failure_rates.csv

## How to apply

Load the in-silico amplicon FASTA sequences produced by primer-extraction of each 16S sub-region together with the Greengenes reference alignment and taxonomy files. Run mothur classify.seqs with the Wang Bayesian classifier against the Greengenes database, sweeping the confidence cutoff parameter across the range 30 to 98 to generate taxonomy assignments at each threshold. Extract the species-level label from each assignment and compare it to the known source species of the reference genome from which the amplicon was derived. Compute the per-region species-level failure rate (fraction of amplicons whose species assignment differs from or fails to reach the source species) for each sub-region and for full-length V1-V9, then write the results to fig1b_species_failure_rates.csv. Rationale: the paper demonstrates that short-read targeting of 16S variable regions cannot achieve the taxonomic resolution of the entire ~1500 bp gene, so the classifier confidence sweep quantifies exactly how much resolution is lost per sub-region.

## Related tools

- **mothur** (Runs classify.seqs with the Wang Bayesian classifier to assign each amplicon a taxonomic lineage against the Greengenes reference database at swept confidence cutoffs)
- **ruffus** (CGAT ruffus pipeline framework used to orchestrate the per-figure analysis pipelines (Figures 1-3 written in Python 2.7), including this classification step) — github.com/cgat-developers/ruffus
- **weinstock_full_length_16s** (Repository containing the code used to perform this in-silico 16S classification analysis reported in Johnson et al.) — github.com/TheJacksonLaboratory/weinstock_full_length_16s

## Examples

```
mothur "#classify.seqs(fasta=in_silico_v4.fasta, reference=gg_13_8_alignment.fasta, taxonomy=gg_13_8_taxonomy.tax, cutoff=80)"
```

## Evaluation signals

- At the relevant confidence cutoff, the computed V4 species-level failure rate reproduces the reported Fig. 1b value of 56%
- Full-length V1-V9 amplicons show near-complete recovery of confident species-level classification relative to all partial sub-regions
- Every sub-region's failure rate is non-zero — the paper states none of the variable regions recaptured the diversity of the full ~1500 bp gene
- fig1b_species_failure_rates.csv contains one failure-rate row per 16S sub-region plus the full-length V1-V9 amplicon
- Sensitivity check: sweeping the classify.seqs cutoff from 30 to 98 yields a monotone relationship between cutoff stringency and species-level failure rate

## Limitations

- Results are classifier- and database-dependent: the Wang Bayesian classifier against Greengenes may behave differently from other classifiers or reference taxonomies
- Sub-region classification accuracy is intrinsically bounded — no partial variable region achieved full-length taxonomic accuracy in this study
- Species-level ground truth depends on correct annotation of the source Greengenes reference genome; mislabeled references propagate as false failures
- Code for Figures 1-3 is written in Python 2.7, which may require a legacy environment

## Evidence

- [abstract] We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene.: "We demonstrate that targeting of 16S variable regions with short-read sequencing platforms cannot achieve the taxonomic resolution afforded by sequencing the entire (~1500 bp) gene."
- [intro] Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the: "Using an in-silico dataset of sequences taken from public databases we show that commonly targeted 16S sub-regions, such as V4, are unable to match the taxonomic accuracy achieved when sequencing the"
- [discussion] none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [readme] The code underlying Figures 1-3 is written in Python 2.7.: "The code underlying Figures 1-3 is written in Python 2.7."
