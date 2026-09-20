---
name: cross-sample-strain-tracking
description: Use when shotgun metagenomes from related hosts (e.g. mother-infant pairs,
  household cohabitants) have been profiled with MetaPhlAn and you need to determine
  whether the same strain of a species — including uncharacterized uSGBs like Lachnospiraceae
  SGB4894 — is shared between samples.
when_to_use_negative:
- The target species lacks sufficiently many species-specific markers detected across
  samples (no reliable consensus marker sequence can be built, so strain clustering
  would be spurious).
- The question is species-level presence/absence or relative abundance profiling rather
  than strain-level resolution — MetaPhlAn 4 profiling alone suffices.
- Samples come from environmental or under-characterized habitats where current uSGBs
  fail to capture the sequences, per the article's discussion of profiling limitations.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3174
tools:
- name: MetaPhlAn 4
  role: Species-level profiling used to scan abundance profiles and identify the samples
    where the target uSGB (e.g. Lachnospiraceae SGB4894) is detected; also provides
    the marker database underlying StrainPhlAn
  repo: https://github.com/biobakery/MetaPhlAn
- name: StrainPhlAn
  role: 'Strain-level microbial population genomics: builds per-sample consensus marker
    sequences and reconstructs the strain phylogeny from species-specific marker alignments'
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Computational tool for species-level microbial profiling from metagenomic
    shotgun sequencing data; StrainPhlAn is available within MetaPhlAn
  repo: https://github.com/biobakery/MetaPhlAn
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:14d0d594842256ef5dce8d9b4f4f68092b7f23d3d55e802ae8872c217d73c796
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - StrainPhlAn
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-sample-strain-tracking

## Summary

Tracks individual microbial strains across host samples by reconstructing strain-level phylogenies from species-specific marker alignments with StrainPhlAn (available within MetaPhlAn), enabling inference of strain sharing and transmission (e.g. mother-to-infant vertical transmission) between hosts.

## When to use

Use when shotgun metagenomes from related hosts (e.g. mother-infant pairs, household cohabitants) have been profiled with MetaPhlAn and you need to determine whether the same strain of a species — including uncharacterized uSGBs like Lachnospiraceae SGB4894 — is shared between samples.

## When NOT to use

- The target species lacks sufficiently many species-specific markers detected across samples (no reliable consensus marker sequence can be built, so strain clustering would be spurious).
- The question is species-level presence/absence or relative abundance profiling rather than strain-level resolution — MetaPhlAn 4 profiling alone suffices.
- Samples come from environmental or under-characterized habitats where current uSGBs fail to capture the sequences, per the article's discussion of profiling limitations.

## Inputs

- MetaPhlAn 4 species-abundance profiles for the sample cohort (to select target-species-positive metagenomes)
- Marker-gene alignments for the target species (species-specific markers extracted from shotgun metagenomes)
- Sample metadata defining host-relationship pairs (e.g. mother-infant dyads, household cohabitation)

## Outputs

- Per-sample consensus marker sequences for the target species
- Strain-level phylogenetic tree in Newick format (e.g. sgb4894_strain_tree.newick)
- Strain-sharing/transmission statistics from tree tip clustering (e.g. transmission counts between relatives, percent shared between cohabitants)

## How to apply

First scan MetaPhlAn 4 species-abundance profiles to select the samples where the target species is detected (e.g. ~5,800 metagenomes positive for Lachnospiraceae SGB4894, including non-human-primate and ancient samples). Then run StrainPhlAn 4 on those samples' marker alignments, retaining the species-specific markers (e.g. 37 SGB4894-specific markers spanning 19,449 nucleotide positions) to build a per-sample consensus marker sequence, and reconstruct the consensus-marker phylogeny exported as Newick (e.g. sgb4894_strain_tree.newick). Finally, compute strain-sharing and transmission statistics from tree tip clustering over the relatedness pairs of interest (e.g. 21 mother-infant case pairs; same-household cohabiting pairs). Correctness is judged by confirming the tree contains all profiled positive samples (e.g. 1,683 SGB4894-positive samples) and that sharing statistics reproduce expected anchors (no mother-infant sharing in 21 cases; 5.63% same-household sharing).

## Related tools

- **MetaPhlAn 4** (Species-level profiling used to scan abundance profiles and identify the samples where the target uSGB (e.g. Lachnospiraceae SGB4894) is detected; also provides the marker database underlying StrainPhlAn) — https://github.com/biobakery/MetaPhlAn
- **StrainPhlAn** (Strain-level microbial population genomics: builds per-sample consensus marker sequences and reconstructs the strain phylogeny from species-specific marker alignments) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Computational tool for species-level microbial profiling from metagenomic shotgun sequencing data; StrainPhlAn is available within MetaPhlAn) — https://github.com/biobakery/MetaPhlAn

## Examples

```
conda install -c bioconda metaphlan; strainphlan -s markers_alignment/*.pkl -m sgb4894_markers.PanTax -t full_tree -o sgb4894_strain_tree.newick
```

## Evaluation signals

- The reconstructed strain tree contains the expected number of profiled positive samples (e.g. 1,683 SGB4894-positive samples) as tips.
- The retained marker set matches the expected species-specific markers (e.g. 37 SGB4894-specific markers spanning 19,449 nucleotide positions).
- Output tree is valid Newick (e.g. sgb4894_strain_tree.newick) and parseable.
- Strain-sharing statistics reproduce reported anchors: no mother-infant strain sharing in all 21 mother-infant case pairs; 5.63% same-household strain sharing.
- Sample selection from MetaPhlAn 4 profiles recovers the expected cohort (e.g. ~5,800 metagenomes where the uSGB was detected).

## Limitations

- Vertical transmission inference depends on reliable detection of the uSGB in both relatives; in the study, only 21 mother-infant cases had reliable detection in both, so transmission rarity estimates are limited by detection rates.
- Current uSGBs do not extensively incorporate viral or eukaryotic microbial sequences due to their unique genomic architectures and quality control requirements.
- Further work remains to better profile under-characterized habitats; environmental and nonhost-associated communities remain highly enriched for sequences not captured by current uSGBs.

## Evidence

- [intro] StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics.: "StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics."
- [discussion] This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa: "This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa"
- [readme] MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
