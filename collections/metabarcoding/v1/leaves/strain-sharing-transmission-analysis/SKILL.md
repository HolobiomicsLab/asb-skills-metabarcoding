---
name: strain-sharing-transmission-analysis
description: Use when MetaPhlAn 4 species-abundance profiles identify samples positive
  for a species (even an uncharacterized uSGB like Lachnospiraceae SGB4894) and the
  goal is strain-level phylogeny plus transmission/sharing statistics across related
  hosts.
when_to_use_negative:
- The species has no reliably detected marker coverage in the target samples — StrainPhlAn
  cannot build per-sample consensus sequences without sufficient SGB-specific markers
- The analysis goal is species-level abundance profiling only; MetaPhlAn 4 profiles
  alone suffice without strain phylogeny
- Samples are environmental or non-host-associated communities, which the article
  notes are 'still highly enriched for sequences not captured even by current uSGBs'
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3293
tools:
- name: StrainPhlAn
  role: Builds per-sample consensus marker sequences from SGB-specific markers and
    reconstructs the strain-level phylogeny
  repo: https://github.com/biobakery/MetaPhlAn
- name: MetaPhlAn 4
  role: Produces the species-abundance profiles used to select SGB-positive samples
    and defines the SGB-specific marker set
  repo: https://github.com/biobakery/MetaPhlAn
- name: SegataLab/pyphlan
  role: Supporting Python utilities for the bioBakery strain-analysis workflow
  repo: https://github.com/SegataLab/pyphlan
- name: conda
  role: Installation channel for MetaPhlAn/StrainPhlAn via bioconda
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41587-023-01688-w
    title: Extending and improving metagenomic taxonomic profiling with uncharacterized
      species using MetaPhlAn 4
  source_content_hash: sha256:87bedae0938d6b2f73a73e8c4baaa4754cbe211f0375ee94286a2f7e8a556449
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/biobakery/MetaPhlAn
  tools:
  - MetaPhlAn
  - MetaPhlAn 4
  - SegataLab/pyphlan
  - StrainPhlAn
  - conda
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3293
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# strain-sharing-transmission-analysis

## Summary

Reconstructs strain-level phylogenies for a species (including uncharacterized SGBs) from MetaPhlAn 4 marker alignments using StrainPhlAn 4, then infers strain sharing and vertical transmission between related hosts (e.g. mother-infant pairs, cohabiting household members) from tree tip clustering.

## When to use

Use when MetaPhlAn 4 species-abundance profiles identify samples positive for a species (even an uncharacterized uSGB like Lachnospiraceae SGB4894) and the goal is strain-level phylogeny plus transmission/sharing statistics across related hosts.

## When NOT to use

- The species has no reliably detected marker coverage in the target samples — StrainPhlAn cannot build per-sample consensus sequences without sufficient SGB-specific markers
- The analysis goal is species-level abundance profiling only; MetaPhlAn 4 profiles alone suffice without strain phylogeny
- Samples are environmental or non-host-associated communities, which the article notes are 'still highly enriched for sequences not captured even by current uSGBs'

## Inputs

- MetaPhlAn 4 species-abundance profiles for the full metagenome cohort (e.g. 19.5k gut metagenomes, 24.5k total)
- Metagenomic shotgun sequencing samples or precomputed marker alignments for SGB-positive samples
- Pairing metadata linking related hosts (mother-infant case pairs, cohabiting household pairs)

## Outputs

- Strain-level phylogenetic tree in Newick format (e.g. sgb4894_strain_tree.newick) with one tip per SGB-positive sample
- Per-sample consensus marker sequences built from retained SGB-specific markers
- Strain-sharing/transmission statistics table (per mother-infant pair and per cohabiting pair)

## How to apply

First scan MetaPhlAn 4 species-abundance profiles to select all metagenomes where the target SGB is detected (e.g. ~5,800 samples for SGB4894, including non-human-primate and ancient samples). Run StrainPhlAn 4 on the samples' marker alignments, retaining the SGB-specific markers (37 markers spanning 19,449 nt positions for SGB4894) to build a per-sample consensus marker sequence. Reconstruct the consensus-marker phylogeny with StrainPhlAn 4 and export it in Newick format (e.g. sgb4894_strain_tree.newick). Compute strain-sharing/transmission statistics from tip clustering: assess mother-infant transmission in paired cases (21 mother-infant pairs for SGB4894) and same-household sharing across all cohabiting pairs. Validate by confirming the tree contains all profiled SGB-positive samples (1,683 for SGB4894) and that sharing statistics reproduce expected anchors (0/21 vertical transmission; 5.63% same-household sharing).

## Related tools

- **StrainPhlAn** (Builds per-sample consensus marker sequences from SGB-specific markers and reconstructs the strain-level phylogeny) — https://github.com/biobakery/MetaPhlAn
- **MetaPhlAn 4** (Produces the species-abundance profiles used to select SGB-positive samples and defines the SGB-specific marker set) — https://github.com/biobakery/MetaPhlAn
- **SegataLab/pyphlan** (Supporting Python utilities for the bioBakery strain-analysis workflow) — https://github.com/SegataLab/pyphlan
- **conda** (Installation channel for MetaPhlAn/StrainPhlAn via bioconda)

## Examples

```
conda install -c bioconda metaphlan
```

## Evaluation signals

- The reconstructed tree contains the full set of profiled SGB-positive samples (e.g. 1,683 samples for SGB4894)
- Retained markers match the expected SGB-specific set (37 markers spanning 19,449 nucleotide positions for SGB4894)
- Mother-infant transmission count reproduces the reported anchor (no strain sharing in 21 mother-infant case pairs)
- Same-household strain-sharing rate matches the reported value (5.63% across cohabiting pairs)
- Output tree parses as valid Newick (sgb4894_strain_tree.newick) with one consensus tip per included sample

## Limitations

- Vertical transmission conclusions depend on reliable detection of the uSGB in both relatives; rare or low-abundance detection limits case counts (21 mother-infant cases for SGB4894)
- Under-characterized habitats remain poorly profiled: 'further work remains to better profile under-characterized habitats'
- The tradeoff between sensitivity and specificity when applying quality control measures to identify real but rare taxa remains an unaddressed downstream statistical challenge

## Evidence

- [intro] StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics.: "StrainPhlAn (available within MetaPhlAn) allows strain-level microbial population genomics."
- [discussion] enabled phylogenetic reconstruction and population genetics inference: "This approach improved the resolution of health-associated biomarkers and enabled phylogenetic reconstruction and population genetics inference for both known and uncharacterized taxa"
- [readme] conda install of MetaPhlAn: "To install MetaPhlan through conda run: $ conda install -c bioconda metaphlan"
- [discussion] under-characterized habitats limitation: "Notably, even with the extended MetaPhlAn 4 SGB and marker set, further work remains to better profile under-characterized habitats."
