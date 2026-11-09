---
name: pcr-replicate-filtering
description: Use when processing raw DADA2 MOTU tables from multi-replicate eDNA metabarcoding
  (e.g. flower samples amplified with 16S rRNA and COI primers) and MOTUs must be
  discarded unless present in ≥2 PCR replicates to reduce false positives.
when_to_use_negative:
- Input is a pre-filtered feature/MOTU table with no per-replicate detection information,
  since the ≥2 PCR replicates rule cannot be applied
- Datasets without PCR replicate structure (single-replicate amplicon runs), where
  this filtering criterion is undefined
- Analyses of non-arthropod targets or markers other than the study's mitochondrial
  16S rRNA and COI primer sets without revalidating the reported read/richness benchmarks
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: Produces the MOTU inference table on which the ≥2 PCR replicates filter is
    applied before taxonomy assignment
- name: tobiasgf/Bioinformatic-tools
  role: Repository of bioinformatic tooling associated with the study's metabarcoding
    workflow
  repo: https://github.com/tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:902ba1a2be192d192ec32eb0567370afefd09926acc98cdfdf7b6da63bc8eae3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - tobiasgf/Bioinformatic-tools
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# pcr-replicate-filtering

## Summary

Filter a DADA2-inferred MOTU table so that MOTUs are retained only when detected in at least two PCR replicates, per the Thomsen & Sigsgaard (2019) flower eDNA protocol, before taxonomy assignment and richness/read-share summarization.

## When to use

Use when processing raw DADA2 MOTU tables from multi-replicate eDNA metabarcoding (e.g. flower samples amplified with 16S rRNA and COI primers) and MOTUs must be discarded unless present in ≥2 PCR replicates to reduce false positives.

## When NOT to use

- Input is a pre-filtered feature/MOTU table with no per-replicate detection information, since the ≥2 PCR replicates rule cannot be applied
- Datasets without PCR replicate structure (single-replicate amplicon runs), where this filtering criterion is undefined
- Analyses of non-arthropod targets or markers other than the study's mitochondrial 16S rRNA and COI primer sets without revalidating the reported read/richness benchmarks

## Inputs

- DADA2-inferred MOTU table per marker (mitochondrial 16S rRNA and COI) with per-PCR-replicate detection flags
- PCR replicate metadata mapping MOTU detections to sample replicates
- Arthropod reference sequence databases for downstream taxonomy assignment (99-100% identity for species-level)

## Outputs

- Replicate-filtered MOTU table (MOTUs detected in ≥2 PCR replicates)
- Per-marker arthropod read totals (e.g. 23,517,933 COI reads; 658,159 16S reads)
- Taxonomic richness counts (species, family, order; expected ≥135 species / 67 families / 14 orders)
- Per-species read-share fractions for dominance quantification

## How to apply

Load the DADA2-inferred MOTU table for the two marker sets targeting mitochondrial 16S rRNA and COI, with per-replicate detection information preserved. For each MOTU, count the number of PCR replicates in which it was detected within a sample and retain it only when detected in at least 2 PCR replicates, matching the study's replicate-filtering rule. After filtering, assign taxonomy against arthropod reference databases (species-level assignment requires a 99-100% sequence-identity match), filter to arthropods, and tally reads per marker — the expected arthropod totals are 23,517,933 COI reads and 658,159 16S reads. The filtered data should support recovery of at least 135 arthropod species in 67 families and 14 orders, with dominance shares such as Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads. Flag any marker whose arthropod read total deviates from these reported figures.

## Related tools

- **DADA2** (Produces the MOTU inference table on which the ≥2 PCR replicates filter is applied before taxonomy assignment)
- **tobiasgf/Bioinformatic-tools** (Repository of bioinformatic tooling associated with the study's metabarcoding workflow) — https://github.com/tobiasgf/Bioinformatic-tools

## Evaluation signals

- MOTU table contains only MOTUs detected in at least 2 PCR replicates
- Computed arthropod read totals per marker match the reported values: 23,517,933 for COI and 658,159 for 16S; flag any deviation
- Taxonomic richness reproduces at least 135 arthropod species, 67 families, and 14 orders
- Per-species read-share fractions reproduce Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads

## Limitations

- The study's own authors note the experiment was not exhaustive, so filtering thresholds may exclude genuine rare taxa and higher richness could be obtained with this eDNA approach
- Species-level taxonomy depends on a 99-100% sequence-identity match against arthropod reference databases, so filtered MOTUs lacking close references cannot be resolved to species
- Reported read-total and richness benchmarks are specific to this study's flower samples and the two marker sets (16S rRNA and COI); they must be re-derived for other datasets

## Evidence

- [abstract] We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders"
- [abstract] two sets of generic primers targeting mitochondrial 16S rRNA and COI: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] terrestrial arthropod species leave traces of DNA on flowers: "Our results show that terrestrial arthropod species leave traces of DNA on the flowers that they interact with."
