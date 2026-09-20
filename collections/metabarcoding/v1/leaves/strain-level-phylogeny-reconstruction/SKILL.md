---
name: strain-level-phylogeny-reconstruction
description: Use when MetaPhlAn 4 species-abundance profiles have detected a species
  (including uncharacterized SGBs) across many shotgun metagenomes and the goal is
  to infer strain-level phylogeny, strain sharing, or host-to-host transmission from
  marker genes.
when_to_use_negative:
- Only species-level taxonomic abundances are needed — plain MetaPhlAn 4 profiling
  suffices without strain-level marker phylogeny
- The target species lacks enough reliably detected samples or species-specific markers
  to support a marker-based consensus phylogeny
- Environmental, nonhost-associated communities dominated by sequences not captured
  by current uSGBs, where marker coverage is expected to be poor
edam_operation: http://edamontology.org/operation_0554
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3697
tools:
- name: StrainPhlAn
  role: Builds per-sample consensus marker sequences and reconstructs the strain-level
    phylogeny from species-specific marker alignments
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn 4
  role: Produces the species-abundance profiles used to select metagenomes where the
    target species (e.g. Lachnospiraceae SGB4894) is detected
  repo: https://github.com/biobakery/MetaPhlAn
- name: PhyloPhlAn
  role: Provides related workflows for phylogenetic characterization of microbial
    taxa
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:9735a104ea5fb600fd3b65f8860759b1bc09585c0f8a2915e8c1666e9dd3a7cc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - PhyloPhlAn
  - StrainPhlAn
  edam_operation: http://edamontology.org/operation_0554
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# strain-level-phylogeny-reconstruction

## Summary

Reconstructs per-sample strain-level phylogenies from metagenomic marker alignments using StrainPhlAn 4, enabling strain sharing and transmission inference (e.g. mother-to-infant vertical transmission) even for uncharacterized species (uSGBs) detected by MetaPhlAn 4.

## When to use

Use when MetaPhlAn 4 species-abundance profiles have detected a species (including uncharacterized SGBs) across many shotgun metagenomes and the goal is to infer strain-level phylogeny, strain sharing, or host-to-host transmission from marker genes.

## When NOT to use

- Only species-level taxonomic abundances are needed — plain MetaPhlAn 4 profiling suffices without strain-level marker phylogeny
- The target species lacks enough reliably detected samples or species-specific markers to support a marker-based consensus phylogeny
- Environmental, nonhost-associated communities dominated by sequences not captured by current uSGBs, where marker coverage is expected to be poor

## Inputs

- MetaPhlAn 4 species-abundance profiles for ~19.5k gut metagenomes (used to select samples where the target species is detected)
- Marker-gene alignments for the target species' markers (e.g. 37 SGB4894-specific markers spanning 19,449 nucleotide positions)
- Sample metadata pairing samples of interest (e.g. mother-infant dyads, household cohabitant pairs)

## Outputs

- Per-sample consensus marker sequences built from the retained species-specific markers
- Strain-level phylogeny in Newick format (e.g. sgb4894_strain_tree.newick) containing all profiled positive samples (1,683 for SGB4894)
- Strain-sharing/transmission statistics derived from tree tip clustering (e.g. 0/21 mother-infant transmission events; 5.63% same-household sharing)

## How to apply

First scan MetaPhlAn 4 species-abundance profiles to select the metagenomes where the target species is detected (in the SGB4894 case, ~5,800 metagenomes including 79 non-human-primate and 12 ancient samples). Then run StrainPhlAn 4 on those samples' marker alignments, retaining only the species-specific markers (37 SGB4894-specific markers spanning 19,449 nucleotide positions) to build a per-sample consensus marker sequence. Reconstruct the consensus-marker phylogeny with StrainPhlAn 4 and export it in Newick format (sgb4894_strain_tree.newick). Finally, compute strain-sharing/transmission statistics from tree tip clustering — e.g. mother-infant transmission over 21 mother-infant case pairs and same-household sharing across cohabiting pairs.

## Related tools

- **StrainPhlAn** (Builds per-sample consensus marker sequences and reconstructs the strain-level phylogeny from species-specific marker alignments) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn 4** (Produces the species-abundance profiles used to select metagenomes where the target species (e.g. Lachnospiraceae SGB4894) is detected) — https://github.com/biobakery/MetaPhlAn
- **PhyloPhlAn** (Provides related workflows for phylogenetic characterization of microbial taxa)

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- The reconstructed Newick tree contains all profiled species-positive samples (1,683 SGB4894-positive samples for the SGB4894 case)
- The retained marker set matches the species-specific marker count and span (37 markers, 19,449 nucleotide positions for SGB4894)
- Tip-clustering statistics reproduce reported anchors: no mother-infant strain sharing in 21 mother-infant case pairs; 5.63% same-household strain sharing
- Tree is exported in valid Newick format (sgb4894_strain_tree.newick) and parseable by standard phylogenetics libraries

## Limitations

- Vertical transmission conclusions depend on the species being reliably detected in both relatives (only 21 SGB4894 mother-infant case pairs were assessable)
- Many related downstream statistical challenges remain, such as the tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa
- MetaPhlAn 4's SGB and marker set still under-captures environmental, nonhost-associated, and under-studied microbial communities, limiting strain phylogenies in those habitats

## Evidence

- [intro] StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics.: "StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics."
- [abstract] Application to >24,500 metagenomes shows uncharacterized species can be profiled at single-strain resolution.: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes"
- [readme] MetaPhlAn provides species-level microbial profiling from metagenomic shotgun sequencing data.: "MetaPhlAn is a computational tool for species-level microbial profiling (bacteria, archaea, eukaryotes, and viruses) from metagenomic shotgun sequencing data."
- [readme] MetaPhlAn is installed via conda from bioconda.: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
