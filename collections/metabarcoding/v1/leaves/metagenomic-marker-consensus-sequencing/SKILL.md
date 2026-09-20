---
name: metagenomic-marker-consensus-sequencing
description: Use when MetaPhlAn 4 profiles detect a target species (including taxonomically
  unidentified SGBs) across many shotgun metagenomes and you need a strain phylogeny
  or strain-sharing/transmission analysis from marker alignments.
when_to_use_negative:
- The target taxon has high-quality reference isolate genomes available, so a reference-based
  phylogeny (e.g., PhyloPhlAn) is preferable to marker consensus reconstruction
- Only a few samples contain the clade, so per-sample marker coverage is too sparse
  for reliable consensus sequence reconstruction
- The question is species-level abundance profiling rather than strain-level population
  genomics — plain MetaPhlAn 4 profiling suffices
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0084
tools:
- name: MetaPhlAn 4
  role: Defines unique marker genes for species-level genome bins (incl. 4,992 unidentified
    uSGBs) and produces the species-abundance profiles used to select target samples
  repo: https://github.com/biobakery/MetaPhlAn
- name: StrainPhlAn
  role: Builds per-sample consensus marker sequences and reconstructs the strain-level
    phylogeny from clade-specific marker alignments
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Species-level microbial profiling of bacteria, archaea, eukaryotes, and viruses
    from shotgun data; StrainPhlAn is available within MetaPhlAn
  repo: https://github.com/biobakery/MetaPhlAn
- name: bioBakery
  role: Integrated platform environment within which the MetaPhlAn/StrainPhlAn profiling
    and phylogeny workflows are run
  repo: https://github.com/biobakery/biobakery
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:2ce8ef8a5726ef2fe05cad9ed7820e65a0d686702dd82a6c4a2050aa5617f9ba
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - StrainPhlAn
  - bioBakery
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# metagenomic-marker-consensus-sequencing

## Summary

Build per-sample consensus sequences from clade-specific marker-gene alignments so that strain-level phylogenies can be reconstructed for species lacking reference isolates (e.g., uncharacterized SGBs profiled by MetaPhlAn 4) using StrainPhlAn.

## When to use

Use when MetaPhlAn 4 profiles detect a target species (including taxonomically unidentified SGBs) across many shotgun metagenomes and you need a strain phylogeny or strain-sharing/transmission analysis from marker alignments.

## When NOT to use

- The target taxon has high-quality reference isolate genomes available, so a reference-based phylogeny (e.g., PhyloPhlAn) is preferable to marker consensus reconstruction
- Only a few samples contain the clade, so per-sample marker coverage is too sparse for reliable consensus sequence reconstruction
- The question is species-level abundance profiling rather than strain-level population genomics — plain MetaPhlAn 4 profiling suffices

## Inputs

- MetaPhlAn 4 species-abundance profile table (per-sample clade detection/abundance)
- Shotgun metagenome samples in which the target clade is detected (~5,800 SGB4894-detected samples, incl. 79 non-human-primate and 12 ancient samples)
- Clade-specific marker alignments (37 SGB4894-specific markers, 19,449 nt)
- Mother-infant and household relationship metadata for transmission statistics

## Outputs

- Per-sample consensus marker sequences for the target clade
- Newick strain phylogeny (sgb4894_strain_tree.newick) over 1,683 SGB4894-positive samples
- Strain-sharing/transmission statistics (e.g., 0/21 mother-infant transmission cases; 5.63% same-household strain sharing)

## How to apply

First, scan MetaPhlAn 4 species-abundance profiles to select the metagenomes where the target clade is detected (for Lachnospiraceae SGB4894 this yielded ~5,800 samples, of which 1,683 were profiled SGB4894-positive). Next, restrict to the clade's unique marker genes (37 SGB4894-specific markers spanning 19,449 nucleotide positions) and build a per-sample consensus marker sequence from their alignments, discarding poorly covered samples. Then reconstruct the consensus-marker phylogeny with StrainPhlAn and export it in Newick format (sgb4894_strain_tree.newick). Finally, derive strain-sharing or transmission statistics from tip clustering (e.g., mother-infant transmission cases and same-household cohabiting pairs). The rationale is that MetaPhlAn 4's markers for SGBs — unique genes defined from 1.01 M reference and metagenome-assembled genomes for 26,970 species-level genome bins — provide the only sequence information available for species with no cultured isolate.

## Related tools

- **MetaPhlAn 4** (Defines unique marker genes for species-level genome bins (incl. 4,992 unidentified uSGBs) and produces the species-abundance profiles used to select target samples) — https://github.com/biobakery/MetaPhlAn
- **StrainPhlAn** (Builds per-sample consensus marker sequences and reconstructs the strain-level phylogeny from clade-specific marker alignments) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Species-level microbial profiling of bacteria, archaea, eukaryotes, and viruses from shotgun data; StrainPhlAn is available within MetaPhlAn) — https://github.com/biobakery/MetaPhlAn
- **bioBakery** (Integrated platform environment within which the MetaPhlAn/StrainPhlAn profiling and phylogeny workflows are run) — https://github.com/biobakery/biobakery

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- The reconstructed tree contains exactly the 1,683 profiled SGB4894-positive samples as tips
- The retained marker set matches the expected clade markers (37 SGB4894-specific markers, 19,449 nucleotide positions)
- Output phylogeny is valid Newick (sgb4894_strain_tree.newick) and is readable by standard tree parsers
- Tip-cluster-based strain sharing reproduces reported anchors: no mother-infant strain sharing in all 21 case pairs, 5.63% same-household strain sharing
- Sample selection from MetaPhlAn 4 profiles matches expected counts (~5,800 detected samples including 79 non-human-primate and 12 ancient samples)

## Limitations

- Consensus markers cannot be built for samples with insufficient marker coverage of the target clade
- Environmental, nonhost-associated, and under-studied communities remain enriched for sequences not captured by current uSGBs
- Further work remains to better profile under-characterized habitats; tradeoffs between sensitivity and specificity in quality control for real but rare taxa remain an unaddressed downstream statistical challenge
- StrainPhlAn results should be cited to both the MetaPhlAn and StrainPhlAn papers per the tool documentation

## Evidence

- [abstract] Here we present MetaPhlAn 4, which integrates information from metagenome assemblies and microbial isolate genomes for more comprehensive metagenomic taxonomic profiling.: "Here we present MetaPhlAn 4, which integrates information from metagenome assemblies and microbial isolate genomes for more comprehensive metagenomic taxonomic profiling."
- [intro] StrainPhlAn allows strain-level microbial population genomics.: "StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics."
- [discussion] Discussion quote on phylogenetic reconstruction for uncharacterized taxa.: "This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa"
- [readme] README description of MetaPhlAn input data type.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
