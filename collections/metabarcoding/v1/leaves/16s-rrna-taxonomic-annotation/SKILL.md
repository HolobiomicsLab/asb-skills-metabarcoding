---
name: 16s-rrna-taxonomic-annotation
description: Use when OTU clusters (e.g. HMP V1V3/V3V5 at 97% PSD) must be taxonomically
  labeled, or when testing whether representative-based annotation mislabels member
  sequences within an OTU.
when_to_use_negative:
- Input is already a feature/OTU table with per-read taxonomy — no reclassification
  of members is needed
- OTUs were not clustered by sequence similarity (the skill targets the representative-inherited-annotation
  failure mode of PSD-based OTUs at 97%)
- Sequences are not 16S rRNA amplicons — the RDP Classifier/Greengenes configuration
  and 16S reference tree placement would be inappropriate
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3293
- http://edamontology.org/topic_0092
tools:
- name: QIIME
  role: Runs the assign_taxonomy step with the RDP Classifier and Greengenes training
    set to annotate representatives and members; also the source pipeline for the
    HMP OTUs
  repo: github.com/namphuon/phylo
- name: SEPP
  role: Inserts each short 16S amplicon read into the fixed reference tree for per-read
    placement
  repo: github.com/namphuon/phylo
- name: pplacer
  role: Maximum-likelihood/Bayesian placement engine that derives per-read taxonomic
    lineages on the fixed reference tree
  repo: github.com/namphuon/phylo
- name: RAxML version 8
  role: Part of the reference-tree construction lineage used to prepare the fixed
    tree for placement
- name: PASTA
  role: Builds the ultra-large 16S multiple sequence alignment underlying the fixed
    reference tree
- name: FastTree
  role: FastTree 2 lineage used for reference-tree construction supporting placement
provenance:
  source_task_ids:
  - task_007
  - task_008
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:2c99645118c481dbcec0ed837a725bb3d5dbeb0e43e52172cba5c62297c422d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastTree
  - PASTA
  - QIIME
  - RAxML version 8
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3293
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# 16S rRNA taxonomic annotation

## Summary

Assigns taxonomy to 16S rRNA reads and OTU representative sequences, and quantifies how often member reads of an OTU disagree with the annotation inherited from its single representative sequence (RDP Classifier/Greengenes via QIIME, or per-read phylogenetic placement with SEPP/pplacer).

## When to use

Use when OTU clusters (e.g. HMP V1V3/V3V5 at 97% PSD) must be taxonomically labeled, or when testing whether representative-based annotation mislabels member sequences within an OTU.

## When NOT to use

- Input is already a feature/OTU table with per-read taxonomy — no reclassification of members is needed
- OTUs were not clustered by sequence similarity (the skill targets the representative-inherited-annotation failure mode of PSD-based OTUs at 97%)
- Sequences are not 16S rRNA amplicons — the RDP Classifier/Greengenes configuration and 16S reference tree placement would be inappropriate

## Inputs

- OTU representative sequence FASTA files (HMP V1V3, V3V5 regions, clustered at 97% PSD)
- OTU member sequence FASTA files grouped by OTU
- Greengenes reference training set (reference sequences plus taxonomy)
- Fixed 16S reference tree and alignment (RAxML/PASTA/FastTree lineage) for phylogenetic placement

## Outputs

- Taxonomic assignments for representative sequences and all OTU member sequences (RDP Classifier/Greengenes)
- annotation_instability.csv with region/stratum/percentage fields (e.g. V3V5 12.9%, V1V3 19.8%; high-MSD strata 13.5% and 26.2%; Lactobacillus V1V3 2.0%)
- per_read_placements.csv: per-read SEPP/pplacer placement lineages with flags for discordance against the OTU representative's inherited annotation

## How to apply

Load the OTU representative and member FASTA files for the 16S region of interest (e.g. HMP V1V3 and V3V5 OTUs clustered by PSD at 97% sequence similarity). Classify each representative sequence with the RDP Classifier through QIIME's assign_taxonomy step using the Greengenes reference training set, then classify every member sequence of each OTU with the identical RDP Classifier/Greengenes configuration. Flag each OTU in which at least one member receives a taxonomic assignment differing from the representative's annotation at any rank, and compute the instability percentage per region and per stratum (overall vs. high-MSD OTUs, average MSD >= 0.03 between member and representative). As an alternative to representative-inherited labels, place every short amplicon read onto a fixed 16S reference tree (RAxML/PASTA/FastTree lineage) using SEPP and pplacer, and compare each read's placement-derived lineage against the annotation inherited from its OTU representative. The rationale is that representative-based annotation propagates a single label to all members even when OTUs are poorly compacted — 80.5% of V3V5 OTUs contained at least one sequence >3% MSD from the representative, motivating per-read classification.

## Related tools

- **QIIME** (Runs the assign_taxonomy step with the RDP Classifier and Greengenes training set to annotate representatives and members; also the source pipeline for the HMP OTUs) — github.com/namphuon/phylo
- **SEPP** (Inserts each short 16S amplicon read into the fixed reference tree for per-read placement) — github.com/namphuon/phylo
- **pplacer** (Maximum-likelihood/Bayesian placement engine that derives per-read taxonomic lineages on the fixed reference tree) — github.com/namphuon/phylo
- **RAxML version 8** (Part of the reference-tree construction lineage used to prepare the fixed tree for placement)
- **PASTA** (Builds the ultra-large 16S multiple sequence alignment underlying the fixed reference tree)
- **FastTree** (FastTree 2 lineage used for reference-tree construction supporting placement)

## Examples

```
assign_taxonomy.py -i otu_rep_set.fasta -r gg_otus.fasta -t gg_taxonomy.txt -m rdp  # QIIME RDP Classifier + Greengenes for representatives; repeat identically for all OTU members
```

## Evaluation signals

- annotation_instability.csv reproduces the reported percentages within rounding tolerance: 12.9% (V3V5), 19.8% (V1V3/V1V5), 13.5% (V3V5 high-MSD), 26.2% (V1V5 high-MSD), 2.0% (Lactobacillus V1V3), and contains region/stratum/percentage fields
- Identical RDP Classifier/Greengenes configuration is used for representatives and members, so discordances are attributable to sequence differences rather than classifier settings
- High-MSD stratum (average MSD >= 0.03 between member and representative) shows instability at or above the overall rate for the same region (13.5% vs 12.9%; 26.2% vs 19.8%)
- per_read_placements.csv covers every read with a placement-derived lineage, and discordance flags are consistent between a read's placement taxonomy and its OTU representative's taxonomy
- Sanity check on OTU divergence: ~80.5% of V3V5 OTUs contain at least one member with MSD > 0.03 from the representative

## Limitations

- The 97% sequence similarity threshold is only a rough approximation for delineating species and can overestimate evolutionary similarity between pairs of sequences
- Pairwise-alignment-based similarity (PSD) can produce poorly clustered OTUs whose members exceed 3% MSD from the representative, so representative-inherited annotations can mislabel a substantial fraction of OTUs
- The field needs to move beyond simple clustering techniques for 16S analysis — representative-based annotation alone is not sufficient for downstream analyses

## Evidence

- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [intro] From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method: "From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [intro] we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold: "we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold"
