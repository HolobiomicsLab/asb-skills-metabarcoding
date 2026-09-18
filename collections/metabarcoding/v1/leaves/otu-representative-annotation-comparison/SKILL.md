---
name: otu-representative-annotation-comparison
description: Use when QIIME-generated 97%-similarity OTUs (e.g. HMP V1V3 or V3V5 read
  collections) are annotated from one representative sequence and you need to quantify
  how often member reads would receive a different taxonomic label.
when_to_use_negative:
- When the goal is de novo OTU clustering itself (PSD/MSD/BLD metric comparison) rather
  than auditing annotation transfer from representative sequences.
- When full-length 16S sequences are available so reads can be classified directly
  without phylogenetic placement onto a short-amplicon reference tree.
- When a feature table with per-feature taxonomy already exists and no representative-sequence
  annotation is being applied to member reads.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0084
tools:
- name: QIIME
  role: Source of the HMP OTU read collections clustered by PSD at 97% similarity
    that are audited
- name: SEPP
  role: SATé-enabled phylogenetic placement of each short amplicon read into the fixed
    16S reference tree
- name: pplacer
  role: Maximum-likelihood/Bayesian placement engine assigning each read a lineage
    on the fixed reference tree
- name: RAxML version 8
  role: Reference-tree construction for the 16S marker
- name: PASTA
  role: Ultra-large multiple sequence alignment for the 16S reference
- name: FastTree 2
  role: Approximate maximum-likelihood tree construction for large 16S alignments
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:047333f5192805aad1882b0c97847af14f57a794b5825b213785d6ae06eb5f25
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - FastTree 2
  - PASTA
  - QIIME
  - RAxML version 8
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.38461538461538464
n_claims: 13
---

# otu-representative-annotation-comparison

## Summary

Audits the standard 16S practice of annotating an OTU from a single representative sequence by placing every member read onto a fixed 16S reference tree (SEPP/pplacer) and comparing per-read placement-derived taxonomy with the annotation each read inherits from its OTU representative.

## When to use

Use when QIIME-generated 97%-similarity OTUs (e.g. HMP V1V3 or V3V5 read collections) are annotated from one representative sequence and you need to quantify how often member reads would receive a different taxonomic label.

## When NOT to use

- When the goal is de novo OTU clustering itself (PSD/MSD/BLD metric comparison) rather than auditing annotation transfer from representative sequences.
- When full-length 16S sequences are available so reads can be classified directly without phylogenetic placement onto a short-amplicon reference tree.
- When a feature table with per-feature taxonomy already exists and no representative-sequence annotation is being applied to member reads.

## Inputs

- QIIME-generated OTU read collections (HMP V1V3 and V3V5, PSD clustering at 97% sequence similarity)
- Representative sequence annotation per OTU (16S classification method output)
- Fixed 16S reference tree and multiple sequence alignment (RAxML 8 / PASTA / FastTree 2)

## Outputs

- per_read_placements.csv (one placed lineage/taxonomic label per amplicon read)
- Per-OTU discordance flags comparing read placement taxonomy with representative-inherited annotation

## How to apply

Load the QIIME HMP OTU read collections (V1V3 and V3V5, clustered by PSD at a 97% sequence similarity threshold) and, for each OTU, record the taxonomic annotation that all member reads inherit from the single representative sequence. Build or obtain a fixed 16S reference tree and alignment using the RAxML version 8 / PASTA / FastTree 2 lineage of reference-tree construction. Use SEPP (SATé-enabled phylogenetic placement) followed by pplacer's maximum-likelihood/Bayesian placement to insert each short amplicon read into the reference tree. Extract each read's placed lineage/taxonomic label into per_read_placements.csv and compare it, rank by rank, with the representative-inherited annotation, flagging discordances per OTU. Rationale: since a representative's annotation is applied to all remaining sequences in the OTU, representative-based labeling can mislabel reads when the OTU is not evolutionarily compact (e.g. reads with MSD>0.03 from the representative).

## Related tools

- **QIIME** (Source of the HMP OTU read collections clustered by PSD at 97% similarity that are audited)
- **SEPP** (SATé-enabled phylogenetic placement of each short amplicon read into the fixed 16S reference tree)
- **pplacer** (Maximum-likelihood/Bayesian placement engine assigning each read a lineage on the fixed reference tree)
- **RAxML version 8** (Reference-tree construction for the 16S marker)
- **PASTA** (Ultra-large multiple sequence alignment for the 16S reference)
- **FastTree 2** (Approximate maximum-likelihood tree construction for large 16S alignments)

## Evaluation signals

- Per-read placement table covers every read in the V1V3 and V3V5 OTU collections with a non-empty placed lineage.
- Reported discordance rate falls in the paper's range: 12.9% of V3V5 OTUs and 19.8% of V1V3 OTUs contain reads annotated differently from the representative.
- Cross-check reads with MSD>0.03 from the representative (paper: 2.7% of all V3V5 reads; 80.5% of V3V5 OTUs had at least one such read) against discordance flags.
- Taxonomic comparison is done at a consistent rank between placement-derived label and representative-inherited annotation.
- Reference tree is fixed and shared between SEPP insertion and pplacer placement so per-read labels are comparable across OTUs.

## Limitations

- 97% 16S rRNA sequence similarity is only a rough approximation for delineating species, so placement-based labels are themselves approximate.
- Percent sequence similarity (PSD) can overestimate evolutionary similarity; OTUs that look compact by PSD can span MSD/BLD >0.03, and 80.5% of V3V5 OTUs had at least one read >3% dissimilar from the representative.
- Short amplicon reads require insertion into a reference tree (SEPP/pplacer), so label quality depends on the reference tree and alignment construction.

## Evidence

- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [abstract] representative annotation workflow: "From each OTU, a single sequence is selected as a representative"
- [results] MSD dissimilarity from representative: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [results] fraction of reads far from representative: "2.7% of all sequences across all the V3V5 OTUs had a MSD >3% from the representative"
- [intro] annotation inheritance in standard pipeline: "From the OTU cluster, a single sequence is selected as a representative sequence. The representative sequence is annotated using a 16S classification method"
