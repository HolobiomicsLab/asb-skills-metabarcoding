---
name: 16s-amplicon-otu-clustering
description: Use when amplicon FASTA reads from 16S variable regions or full-length
  gene must be dereplicated and clustered into OTUs at a chosen identity threshold
  (97/98/99%) to quantify taxa such as Bacteroides at genus/species level.
when_to_use_negative:
- Input is already a feature/OTU table or abundance matrix rather than raw/dereplicated
  amplicon reads
- The goal is resolving intragenomic 16S copy variants at strain level via SNP phasing
  rather than OTU-level abundance estimation
- Assessing whether a partial sub-region matches full-gene taxonomic accuracy when
  only the V4 region is available — the article shows V4 performed worst and no partial
  region recaptures full diversity even at 99% identity
edam_operation: http://edamontology.org/operation_0233
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: USEARCH
  role: Dereplication and OTU clustering at 97/98/99% identity via cluster_otus with
    chimera detection disabled, plus alignment of OTU representatives to the RTG 16S
    reference database for species-level taxonomy
- name: PacBio
  role: Platform generating full-length (V1-V9) 16S amplicon reads suitable for 99%-identity
    species-level OTU clustering
- name: ruffus
  role: CGAT ruffus pipeline framework used to organize the analysis pipelines underlying
    Figures 1-3 in the weinstock_full_length_16s repository
  repo: https://github.com/cgat-developers/ruffus
merged_aliases:
- otu-clustering-of-16s-amplicons
merged_alias_records:
- alias: otu-clustering-of-16s-amplicons
  slug: otu-clustering-of-16s-amplicons
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1038/s41467-019-13036-1
    title: Evaluation of 16S rRNA gene sequencing for species and strain-level microbiome
      analysis
  source_content_hash: sha256:6f8b289a9b0e60ac395df8f08c29bd14e94e1a3aed782f87ad568907a72dc193
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cgat-developers/ruffus
  tools:
  - PacBio
  - USEARCH
  - ruffus
  edam_operation: http://edamontology.org/operation_0233
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# 16s-amplicon-otu-clustering

## Summary

Cluster 16S rRNA amplicon reads (full-length V1-V9 PacBio or partial V1-V3 MiSeq) into OTUs at high identity thresholds (97%, 98%, 99%) with USEARCH, then assign species-level taxonomy by aligning OTU representatives to a curated 16S reference database, enabling species-level Bacteroides quantification comparable to mWGS.

## When to use

Use when amplicon FASTA reads from 16S variable regions or full-length gene must be dereplicated and clustered into OTUs at a chosen identity threshold (97/98/99%) to quantify taxa such as Bacteroides at genus/species level.

## When NOT to use

- Input is already a feature/OTU table or abundance matrix rather than raw/dereplicated amplicon reads
- The goal is resolving intragenomic 16S copy variants at strain level via SNP phasing rather than OTU-level abundance estimation
- Assessing whether a partial sub-region matches full-gene taxonomic accuracy when only the V4 region is available — the article shows V4 performed worst and no partial region recaptures full diversity even at 99% identity

## Inputs

- Preprocessed 16S amplicon read set(s) as FASTA (PacBio V1-V9 full-length and/or MiSeq V1-V3) from the weinstock_full_length_16s repository
- In-silico 16S sub-region amplicon FASTA files (e.g., from a prior extraction task)
- Curated 16S reference database (RTG) for taxonomy assignment
- Reference distinct full-length 16S sequence counts for diversity comparison

## Outputs

- OTU tables (per platform/region at each identity threshold) with abundance counts
- otu_counts.csv compiling per-region, per-threshold OTU counts
- Bacteroides genus- and species-level relative abundance profiles per sample
- Taxonomy assignments for OTU representative sequences (species level)

## How to apply

Load the preprocessed amplicon read set (e.g., PacBio V1-V9 full-length or MiSeq V1-V3 FASTA from github.com/TheJacksonLaboratory/weinstock_full_length_16s), dereplicate, and run USEARCH cluster_otus at the chosen identity threshold (-id 0.97, -id 0.98, or -id 0.99) with chimera detection disabled (no -uchime_denovo step). Record OTU counts per region/threshold and compare against the number of distinct full-length sequences in the source database — per the article, no partial sub-region (worst: V4) recaptured full diversity even at 99% identity, so full-length reads clustered at 99% are preferred for species-level resolution. Align OTU representative sequences to the RTG reference 16S database with USEARCH to assign species-level taxonomy, then compute genus- and species-level relative abundance profiles from the OTU tables. Full-length 16S clustered at 99% identity should give Bacteroides species relative abundances that reasonably estimate mWGS-based quantification and preserve high- vs low-Bacteroides sample grouping.

## Related tools

- **USEARCH** (Dereplication and OTU clustering at 97/98/99% identity via cluster_otus with chimera detection disabled, plus alignment of OTU representatives to the RTG 16S reference database for species-level taxonomy)
- **PacBio** (Platform generating full-length (V1-V9) 16S amplicon reads suitable for 99%-identity species-level OTU clustering)
- **ruffus** (CGAT ruffus pipeline framework used to organize the analysis pipelines underlying Figures 1-3 in the weinstock_full_length_16s repository) — https://github.com/cgat-developers/ruffus

## Examples

```
usearch -cluster_otus reads.fa -id 0.99 -otus otus.fa -uparseout uparse.txt
```

## Evaluation signals

- At 99% identity, no partial sub-region OTU count equals the number of distinct full-length sequences in the source database; V4 should show the largest shortfall (reproduces Fig. 1d / Supplementary Fig. 3)
- Full-length V1-V9 99%-OTU Bacteroides species relative abundances provide reasonable estimates of matched mWGS-measured species abundances (Fig. 3a-b)
- High- vs low-Bacteroides sample grouping derived from full-length 16S OTU profiles matches the mWGS-based grouping per individual
- Bacteroides intestinalis, abundant in mWGS and full-length data, is absent from the V1-V3 MiSeq profile for the relevant sample
- Reproducible OTU counts are recorded per region and threshold in otu_counts.csv with chimera detection disabled

## Limitations

- V1-V3 MiSeq sequencing failed to detect the abundant Bacteroides intestinalis in one sample, so partial-region clustering can miss species entirely
- No partial variable region recaptured the diversity represented by the full ~1500 bp gene, even at 99% identity; the V4 region performed worst
- PacBio CCS deletion errors coincident with homopolymer runs limit the ability to resolve highly similar sequences, so substitutions rather than insertions/deletions are the reliable signal
- The 99% identity threshold's ability to separate intragenomic vs. inter-genomic variation was not determined for all bacterial species

## Evidence

- [discussion] full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance: "full-length 16S gene sequences clustered at 99% identity provided reasonable estimates of Bacteroides species relative abundance"
- [discussion] none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene: "none of the variable regions covered by partial 16S sequencing were able to recapture the diversity represented when sequencing the full ~1500 bp gene"
- [readme] Code is organised into a set of ruffus pipelines pertaining to different figures.: "Code is organised into a set of [ruffus](http://www.ruffus.org.uk/) pipelines pertaining
to different figures."
- [results] When clustering OTUs at 99% identity, both V1–V9 and V1–V3 sequencing were able to reflect the species-level Bacteroides variation observed by mWGS: [evidence span withheld — anchor status: not_found]
