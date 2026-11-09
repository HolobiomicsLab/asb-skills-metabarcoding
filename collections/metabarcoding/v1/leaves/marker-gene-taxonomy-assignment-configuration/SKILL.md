---
name: marker-gene-taxonomy-assignment-configuration
description: Use when assigning taxonomy to (M)OTUs/ASVs from eDNA metabarcoding of
  16S/18S rRNA, ITS, or COI marker genes in PEMA, and when deciding between alignment-based
  and phylogeny-based assignment for 16S rRNA.
when_to_use_negative:
- The data are shotgun metagenomes rather than amplicon (marker-gene) reads — PEMA
  is designed for metabarcoding amplicon analysis of 16S/18S rRNA, 12S rRNA, ITS,
  and COI.
- A taxonomy table already exists and only downstream ecological analysis (e.g., phyloseq)
  is needed.
- The target marker gene has no OTU route in PEMA — e.g., COI supports only Swarm
  ASV inference, and 12S rRNA (added in v.2.1.4) has no VSEARCH OTU option.
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0091
tools:
- name: PEMA
  role: Containerized pipeline whose Part 3 performs taxonomy assignment and whose
    parameters.tsv selects classifier/database per marker gene
  repo: https://github.com/hariszaf/pema
- name: BDS
  role: BigDataScript language in which PEMA is implemented; checkpoint files enable
    partial re-execution when only the assignment step changes
- name: CREST LCAClassifier
  role: Alignment-based taxonomy assignment for 16S and 18S rRNA against the Silva
    database and for ITS against Unite
- name: RDPClassifier
  role: Classifies COI MOTUs against the MIDORI reference database
- name: RAxML-ng
  role: Builds the 1,000-taxon SILVA_132_SSURef reference tree for the phylogeny-based
    16S rRNA assignment route
- name: EPA-ng
  role: Places reads onto the 1,000-taxon reference tree in the phylogeny-based 16S
    rRNA route
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/gigascience/giaa022
    title: 'PEMA: a flexible Pipeline for Environmental DNA Metabarcoding Analysis
      of the 16S/18S ribosomal RNA, ITS, and COI marker genes'
  source_content_hash: sha256:78f15266d10899a2a60b1a57edd6bf9855bdeb0c77fe159650232f23ba514f98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/hariszaf/pema
  tools:
  - BDS
  - CREST LCAClassifier
  - EPA-ng
  - PEMA
  - RAxML-ng
  - RDPClassifier
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# marker-gene-taxonomy-assignment-configuration

## Summary

Configure taxonomy assignment within PEMA by pairing each marker gene (16S/18S rRNA, ITS, COI) with its alignment-based classifier and reference database, and optionally invoking the phylogeny-based route for 16S rRNA using a Silva-derived reference tree.

## When to use

Use when assigning taxonomy to (M)OTUs/ASVs from eDNA metabarcoding of 16S/18S rRNA, ITS, or COI marker genes in PEMA, and when deciding between alignment-based and phylogeny-based assignment for 16S rRNA.

## When NOT to use

- The data are shotgun metagenomes rather than amplicon (marker-gene) reads — PEMA is designed for metabarcoding amplicon analysis of 16S/18S rRNA, 12S rRNA, ITS, and COI.
- A taxonomy table already exists and only downstream ecological analysis (e.g., phyloseq) is needed.
- The target marker gene has no OTU route in PEMA — e.g., COI supports only Swarm ASV inference, and 12S rRNA (added in v.2.1.4) has no VSEARCH OTU option.

## Inputs

- PEMA parameters.tsv (completed, version-matched to the PEMA release, e.g., v.2.1.4)
- (M)OTU / ASV sequences produced by Part 2 (VSEARCH clustering, CROP, or Swarm v2)
- Reference databases bundled in the container: Silva 128/132, Unite, MIDORI_LONGEST_1.1, SILVA_132_SSURef-derived 1,000-taxon reference tree

## Outputs

- Taxonomy strings per (M)OTU/ASV merged into the final OTU/ASV table with per-sample abundances
- Per-step output subfolders and BDS checkpoint files enabling partial re-execution of taxonomy assignment

## How to apply

In the PEMA parameters.tsv file, select the assignment route per marker gene: CREST LCAClassifier with the Silva database (versions 128 or 132) for 16S and 18S rRNA, the Unite database for ITS, and RDPClassifier with the MIDORI database (e.g., MIDORI_LONGEST_1.1) for COI. For 16S rRNA, optionally enable the phylogeny-based route, which places reads on a 1,000-taxon reference tree built from SILVA_132_SSURef using RAxML-ng, PaPaRa, and EPA-ng. Run the pipeline via its BDS script; checkpoint files and per-step subfolders mean that changing only the assignment algorithm or parameters triggers partial re-execution of downstream steps rather than a full rerun. Assignment results are merged into the final OTU- or ASV-table with taxonomies and per-sample abundances.

## Related tools

- **PEMA** (Containerized pipeline whose Part 3 performs taxonomy assignment and whose parameters.tsv selects classifier/database per marker gene) — https://github.com/hariszaf/pema
- **BDS** (BigDataScript language in which PEMA is implemented; checkpoint files enable partial re-execution when only the assignment step changes)
- **CREST LCAClassifier** (Alignment-based taxonomy assignment for 16S and 18S rRNA against the Silva database and for ITS against Unite)
- **RDPClassifier** (Classifies COI MOTUs against the MIDORI reference database)
- **RAxML-ng** (Builds the 1,000-taxon SILVA_132_SSURef reference tree for the phylogeny-based 16S rRNA assignment route)
- **EPA-ng** (Places reads onto the 1,000-taxon reference tree in the phylogeny-based 16S rRNA route)

## Examples

```
docker run -it -v /<path_to_analysis_directory>/:/mnt/analysis hariszaf/pema; ./pema_latest.bds
```

## Evaluation signals

- The output OTU/ASV table contains taxonomy assignments and per-sample abundances for all marker genes run.
- The reference database chosen matches the marker gene: Silva (128 or 132) for 16S/18S, Unite for ITS, MIDORI for COI.
- For the phylogeny-based 16S route, the reference tree is derived from SILVA_132_SSURef with 1,000 taxa and uses RAxML-ng, PaPaRa, and EPA-ng.
- Changing only the assignment algorithm/parameters in parameters.tsv triggers partial re-execution via BDS checkpoints, not a complete rerun.
- The analysis directory contains the parameters.tsv version matching the PEMA version used (e.g., pema v.2.1.4 with the v.2.1.4 parameters file).

## Limitations

- Taxonomy assignment for COI requires a minimum of 20 GB of RAM (stated in the README).
- Reference database coverage gaps occur — e.g., for B. leachii no entry exists in MIDORI_LONGEST_1.1, so such taxa remain unassigned.
- Taxonomy assignment routes are marker-gene specific: the phylogeny-based route is available only for 16S rRNA, and COI has no VSEARCH OTU option (Swarm ASV only).

## Evidence

- [methods] For the COI marker gene, the RDPClassifier [25] and the MIDORI database [26] are used for the taxonomic assignment: "For the COI marker gene, the RDPClassifier [25] and the MIDORI database [26] are used for the taxonomic assignment"
- [methods] phylogeny-based assignment is available for 16S rRNA marker gene data: "phylogeny-based assignment is available for 16S rRNA marker gene data"
- [readme] For the case of the 16S rRNA marker gene, PEMA includes two separate approaches for taxonomy assignment: alignment-based and phylogenetic-based.: [evidence span withheld — anchor status: not_found]
- [readme] PEMA processes the reads from each sample and returns an OTU- or an ASV-table with the taxonomies of the taxa found and their abundances in each sample.: [evidence span withheld — anchor status: not_found]
- [methods] For B. leachii no entry exists in the MIDORI database, version MIDORI_LONGEST_1.1: "For B. leachii no entry exists in the MIDORI database, version MIDORI_LONGEST_1.1"
