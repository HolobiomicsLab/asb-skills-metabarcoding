---
name: hmp-amplicon-read-processing
description: Use when handling HMP V1V3 or V3V5 16S amplicon OTU read collections
  (QIIME/PSD, 97% threshold) that need per-read phylogenetic placement, per-OTU reference-based
  MSAs, or MSD/BLD comparison of member sequences against OTU representatives.
when_to_use_negative:
- Input is raw, unclustered amplicon reads with no OTU representative structure —
  the skill presumes QIIME/PSD OTUs at the 97% threshold already exist
- Input is already a per-read taxonomy table or an OTU feature table rather than OTU
  member sequences — placement would be redundant
- The analysis goal is de novo OTU clustering from scratch rather than evaluating/annotating
  pre-existing HMP OTUs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_0092
tools:
- name: QIIME
  role: Pipeline that generated the HMP V1V3/V3V5 OTU read collections (PSD, 97% similarity
    threshold) used as the skill's input
- name: SEPP
  role: SATé-enabled phylogenetic placement used by UPP 2.0 to insert short amplicon
    reads/fragmentary sequences into the fixed 16S reference tree and PASTA reference
    alignment
- name: pplacer
  role: Maximum-likelihood/Bayesian placement engine placing each individual read
    onto the fixed reference tree for per-read taxonomic assignment
- name: PASTA
  role: Produces the ultra-large 16S reference alignment that serves as the fixed
    backbone for UPP insertion of OTU member sequences
- name: RAxML version 8
  role: Reference-based tree construction tool in the lineage used to build the fixed
    16S reference tree for placement
- name: FastTree
  role: Approximately-ML tree building in the reference-tree construction lineage
    backing the fixed placement tree
- name: phylo
  role: Paper authors' repository accompanying the OTU-clustering/dissimilarity analyses
  repo: github.com/namphuon/phylo
merged_aliases:
- hmp-amplicon-alignment-processing
merged_alias_records:
- alias: hmp-amplicon-alignment-processing
  slug: hmp-amplicon-alignment-processing
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:c17e029aa5b05bdbf40c91e8be0ce18fe735a7f46c1d4bdb99e9ce4b7f25b723
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - FastTree
  - PASTA
  - QIIME
  - RAxML version 8
  - SEPP
  - phylo
  - pplacer
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.375
n_claims: 8
---

# hmp-amplicon-read-processing

## Summary

Processes Human Microbiome Project (HMP) 16S V1V3 and V3V5 amplicon reads pre-clustered into OTUs at a 97% sequence similarity threshold (PSD), by aligning member sequences to a PASTA reference alignment with UPP/SEPP and placing individual reads onto a fixed reference tree with pplacer. This enables per-read taxonomy extraction and MSD-based assessment of how well each OTU's representative sequence stands in for its members.

## When to use

Use when handling HMP V1V3 or V3V5 16S amplicon OTU read collections (QIIME/PSD, 97% threshold) that need per-read phylogenetic placement, per-OTU reference-based MSAs, or MSD/BLD comparison of member sequences against OTU representatives.

## When NOT to use

- Input is raw, unclustered amplicon reads with no OTU representative structure — the skill presumes QIIME/PSD OTUs at the 97% threshold already exist
- Input is already a per-read taxonomy table or an OTU feature table rather than OTU member sequences — placement would be redundant
- The analysis goal is de novo OTU clustering from scratch rather than evaluating/annotating pre-existing HMP OTUs

## Inputs

- QIIME-generated HMP V1V3 and V3V5 OTU read collections (clustered by PSD at a 97% sequence similarity threshold)
- PASTA-generated 16S reference alignment (fixed backbone alignment)
- Fixed 16S reference tree (RAxML version 8 / FastTree 2 lineage of reference-tree construction)
- Per-OTU representative sequence annotations

## Outputs

- One FASTA-format multiple sequence alignment per OTU, with all member sequences present at the reference alignment length (UPP output, basis for MSD computation)
- per_read_placements.csv: per-read placement table mapping each read to its placed lineage/taxonomic label from the reference tree
- Discordance flags comparing each read's placement-derived taxonomy with the annotation inherited from its OTU representative

## How to apply

Load the QIIME-generated HMP OTU read collections (V1V3 and V3V5, clustered by PSD at a 97% sequence similarity threshold) together with a fixed 16S reference alignment and tree built with the PASTA/RAxML 8/FastTree 2 lineage of tools. For per-OTU MSAs, run UPP 2.0 (the SEPP-based procedure for fragmentary sequences) to insert each OTU's member sequences into the PASTA reference alignment, writing one FASTA-format MSA per OTU at full reference-alignment length; compute the MSA-based sequence dissimilarity (MSD) on these alignments, flagging sequences with MSD>0.03 from the representative. For per-read taxonomy, run SEPP followed by pplacer's maximum-likelihood/Bayesian placement to insert each short amplicon read into the reference tree, then extract each read's placed lineage label into per_read_placements.csv and compare it against the annotation the read would inherit from its OTU's single representative sequence. The rationale is that representative-based annotation is error-prone: 12.9% of V3V5 OTUs and 19.8% of V1V3 OTUs contained reads whose annotations differed from the representative, and 80.5% of V3V5 OTUs had at least one sequence with MSD>0.03 from the representative.

## Related tools

- **QIIME** (Pipeline that generated the HMP V1V3/V3V5 OTU read collections (PSD, 97% similarity threshold) used as the skill's input)
- **SEPP** (SATé-enabled phylogenetic placement used by UPP 2.0 to insert short amplicon reads/fragmentary sequences into the fixed 16S reference tree and PASTA reference alignment)
- **pplacer** (Maximum-likelihood/Bayesian placement engine placing each individual read onto the fixed reference tree for per-read taxonomic assignment)
- **PASTA** (Produces the ultra-large 16S reference alignment that serves as the fixed backbone for UPP insertion of OTU member sequences)
- **RAxML version 8** (Reference-based tree construction tool in the lineage used to build the fixed 16S reference tree for placement)
- **FastTree** (Approximately-ML tree building in the reference-tree construction lineage backing the fixed placement tree)
- **phylo** (Paper authors' repository accompanying the OTU-clustering/dissimilarity analyses) — github.com/namphuon/phylo

## Examples

```
run_sepp.py -t ref.tre -a ref.aln -f hmp_v3v5_reads.fna -c 10000 -n per_read_placement
```

## Evaluation signals

- Every OTU yields a complete UPP alignment file with all member sequences present at the reference alignment length (no missing members, no truncated columns)
- per_read_placements.csv contains exactly one placed-lineage row per input read — no reads silently dropped by SEPP/pplacer
- Flagged discordance counts fall in plausible ranges given the paper's findings: 12.9% of V3V5 OTUs and 19.8% of V1V3 OTUs contain reads whose annotations differ from the representative
- MSD distributions reproduce the paper's pattern (80.5% of V3V5 OTUs with at least one sequence >3% dissimilar from the representative; 2.7% of all sequences with MSD>0.03)
- Reference tree and alignment remain fixed throughout — no re-estimation of the backbone between SEPP and pplacer runs

## Limitations

- 97% 16S rRNA sequence similarity is only a rough approximation for delineating species, so PSD-97% OTUs are intrinsically loose units
- Pairwise sequence alignment-based similarity computation can result in poorly clustered OTUs, which propagates into the representative-sequence annotation being evaluated
- Representative-based annotation mislabels a substantial fraction of OTUs (12.9% V3V5, 19.8% V1V3), so inherited per-read labels must not be trusted without placement comparison

## Evidence

- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [abstract] As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [intro] we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold: "we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold"
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [abstract] using results obtained from the Human Microbiome Project: "using results obtained from the Human Microbiome Project"
