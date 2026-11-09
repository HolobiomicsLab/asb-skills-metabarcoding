---
name: uncharacterized-species-strain-profiling
description: Use when shotgun metagenomes contain an uncharacterized species-level
  genome bin (uSGB) detected by MetaPhlAn 4 and you need strain-level resolution —
  phylogeny reconstruction, strain sharing, or transmission inference — without a
  cultured isolate reference.
when_to_use_negative:
- The species of interest has a cultured isolate reference genome — a reference-based
  strain analysis may be more appropriate than uSGB marker-based profiling.
- The target uSGB is not reliably detected by MetaPhlAn 4 in enough samples to support
  marker alignment and phylogeny (too few positive samples).
- The goal is functional profiling (genes/pathways) rather than taxonomic/strain-level
  profiling — use HUMAnN instead.
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaPhlAn 4
  role: Species-level taxonomic profiling that defines unique markers for 26,970 SGBs
    (4,992 uncharacterized), used to detect the target uSGB across metagenomes and
    select samples for strain analysis
  repo: https://github.com/biobakery/MetaPhlAn
- name: StrainPhlAn
  role: Strain-level microbial population genomics; builds the per-sample consensus
    marker sequences and reconstructs the uSGB strain phylogeny from species-specific
    markers
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn
  role: Installed via conda; core profiling engine invoked within the workflow, citation
    required alongside StrainPhlAn
  repo: https://github.com/biobakery/MetaPhlAn
- name: PhyloPhlAn
  role: Provides specific workflows for phylogenetic characterization supporting the
    phylogenetic reconstruction step
- name: conda
  role: Installation channel for MetaPhlAn (conda install -c bioconda metaphlan)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:39d1d82f52ca867c55d5adf2e9b755e1ff6ad01b9f414fdf40648e0c32d18e04
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - PhyloPhlAn
  - StrainPhlAn
  - conda
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# uncharacterized-species-strain-profiling

## Summary

Profiles and performs strain-level phylogenetic analysis of taxonomically uncharacterized species (uSGBs) by combining MetaPhlAn 4 species detection with StrainPhlAn 4 marker-based consensus tree reconstruction, enabling strain-sharing and transmission inference (e.g. mother-to-infant vertical transmission of Lachnospiraceae SGB4894).

## When to use

Use when shotgun metagenomes contain an uncharacterized species-level genome bin (uSGB) detected by MetaPhlAn 4 and you need strain-level resolution — phylogeny reconstruction, strain sharing, or transmission inference — without a cultured isolate reference.

## When NOT to use

- The species of interest has a cultured isolate reference genome — a reference-based strain analysis may be more appropriate than uSGB marker-based profiling.
- The target uSGB is not reliably detected by MetaPhlAn 4 in enough samples to support marker alignment and phylogeny (too few positive samples).
- The goal is functional profiling (genes/pathways) rather than taxonomic/strain-level profiling — use HUMAnN instead.

## Inputs

- Shotgun metagenomic sequencing datasets (e.g. 19.5k human gut metagenomes, plus non-human-primate and ancient samples)
- MetaPhlAn 4 species-abundance profiles used to select samples where the target uSGB is detected
- SGB-specific marker gene alignments (e.g. 37 SGB4894-specific markers spanning 19,449 nt)
- Metadata pairing samples for transmission analysis (mother-infant case pairs, same-household cohabiting pairs)

## Outputs

- Per-sample consensus marker sequences for the target uSGB
- Strain-level phylogenetic tree in Newick format (sgb4894_strain_tree.newick)
- Strain-sharing/transmission statistics (e.g. 0/21 mother-infant transmission events; 5.63% same-household strain sharing)

## How to apply

First run MetaPhlAn 4 species-abundance profiling across the metagenome collection and filter to samples where the target uSGB (e.g. Lachnospiraceae SGB4894) is reliably detected — in the source workflow ~5,800 of 19,500+ gut metagenomes qualified, including 79 non-human-primate and 12 ancient samples. Next run StrainPhlAn 4 on those samples' marker alignments, retaining only the species-specific markers (37 SGB4894-specific markers spanning 19,449 nucleotide positions) to build a per-sample consensus marker sequence. Reconstruct the consensus-marker phylogeny with StrainPhlAn 4 and export it in Newick format (e.g. sgb4894_strain_tree.newick). Finally compute strain-sharing/transmission statistics from tree tip clustering, e.g. mother-infant transmission in case pairs and same-household cohabiting pairs, validating that the tree contains all profiled positive samples (1,683 SGB4894-positive samples) and that statistics match reported anchors.

## Related tools

- **MetaPhlAn 4** (Species-level taxonomic profiling that defines unique markers for 26,970 SGBs (4,992 uncharacterized), used to detect the target uSGB across metagenomes and select samples for strain analysis) — https://github.com/biobakery/MetaPhlAn
- **StrainPhlAn** (Strain-level microbial population genomics; builds the per-sample consensus marker sequences and reconstructs the uSGB strain phylogeny from species-specific markers) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn** (Installed via conda; core profiling engine invoked within the workflow, citation required alongside StrainPhlAn) — https://github.com/biobakery/MetaPhlAn
- **PhyloPhlAn** (Provides specific workflows for phylogenetic characterization supporting the phylogenetic reconstruction step)
- **conda** (Installation channel for MetaPhlAn (conda install -c bioconda metaphlan))

## Examples

```
conda install -c bioconda metaphlan; metaphlan metagenome.fastq --input_type fastq --bowtie2db metaphlan_databases/mpa_vOct22_CHOCOPhlAnSGB_202212 -o profile.txt; strainphlan -s marker_alignment_dir -m sgb4894_markers -o sgb4894_strain_tree.newick
```

## Evaluation signals

- The reconstructed strain tree contains the expected number of profiled SGB-positive samples (e.g. 1,683 SGB4894-positive samples).
- Tree tip clustering reproduces reported strain-sharing anchors: no mother-infant strain sharing in 21 case pairs and 5.63% same-household strain sharing.
- The retained marker set matches the expected scale (e.g. 37 SGB4894-specific markers spanning 19,449 nucleotide positions).
- Output phylogeny is valid Newick (sgb4894_strain_tree.newick) with one tip per profiled sample.
- MetaPhlAn 4 profiling explains the expected fraction of reads (~20% more in most human gut microbiomes, >40% in less-characterized environments such as the rumen).

## Limitations

- Vertical/strain transmission estimates depend on reliable detection of the uSGB in both relatives; here transmission was absent in all 21 mother-infant cases, but low detection rates limit statistical power.
- Environmental, nonhost-associated, and under-studied microbial communities remain highly enriched for sequences not captured by current uSGBs.
- A sensitivity/specificity tradeoff persists when applying quality control to identify real but rare taxa; further work remains to better profile under-characterized habitats.
- Current methods do not extensively incorporate viral or eukaryotic microbial sequences due to their unique genomic architectures and QC requirements.

## Evidence

- [abstract] MetaPhlAn 4 defines unique marker genes for 26,970 species-level genome bins from a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, 4,992 of which are taxonomically unidentified: "From a curated collection of 1.01 M prokaryotic reference and metagenome-assembled genomes, we define unique marker genes for 26,970 species-level genome bins, 4,992 of them taxonomically"
- [intro] StrainPhlAn enables strain-level population genomics: "StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics."
- [abstract] Application to >24,500 metagenomes shows uncharacterized species can be profiled at single-strain resolution: "Application of the method to >24,500 metagenomes highlights previously undetected species to be strong biomarkers for host conditions and lifestyles in human and mouse microbiomes and shows that even"
- [discussion] Improved resolution enabled phylogenetic reconstruction for uncharacterized taxa: "This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa"
- [readme] Install MetaPhlAn via conda: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
