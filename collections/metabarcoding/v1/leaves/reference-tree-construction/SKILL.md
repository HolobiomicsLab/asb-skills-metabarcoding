---
name: reference-tree-construction
description: Use when short 16S amplicon reads (e.g. HMP V1V3/V3V5) clustered into
  OTUs by PSD at 97% sequence similarity need per-read phylogenetic placement on a
  fixed reference tree to test representative-based annotation.
when_to_use_negative:
- When the input is already a feature/OTU table without the underlying sequence reads
  — placement requires individual read sequences
- When the analysis goal is OTU clustering itself rather than auditing representative-based
  OTU annotation (use PSD/MSD/BLD dissimilarity analyses instead)
- When reads are full-length 16S sequences that can be aligned directly into the reference
  alignment without fragment-aware placement
edam_operation: http://edamontology.org/operation_0323
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3300
tools:
- name: SEPP
  role: Inserts short 16S amplicon reads into the fixed reference tree (SATé-enabled
    phylogenetic placement) so fragment reads can be placed on a full-length backbone
  repo: github.com/namphuon/phylo
- name: pplacer
  role: Maximum-likelihood/Bayesian placement engine that positions each sequence
    onto the fixed reference tree to derive per-read taxonomic labels
- name: RAxML version 8
  role: Phylogenetic analysis tool in the reference-tree construction lineage for
    large 16S phylogenies
- name: PASTA
  role: Ultra-large multiple sequence alignment for nucleotide sequences used to build
    the 16S reference alignment
- name: FastTree 2
  role: Approximately maximum-likelihood tree inference for large alignments in the
    reference-tree construction lineage
- name: QIIME
  role: Source of the HMP OTU read collections clustered by PSD at 97% sequence similarity
    that feed into the placement analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:b5c38af4a9972cbc0a482f7e72082997528512ca447b56fa801be0944c8b9af1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - FastTree 2
  - PASTA
  - QIIME
  - RAxML version 8
  - SEPP
  - pplacer
  edam_operation: http://edamontology.org/operation_0323
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3300
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# reference-tree-construction

## Summary

Build a fixed 16S rRNA reference tree and alignment using the RAxML/PASTA/FastTree 2 lineage of reference-tree construction, then use it as the backbone for short-amplicon read placement via SEPP and pplacer to evaluate per-read taxonomic assignments inside 97% PSD OTUs.

## When to use

Use when short 16S amplicon reads (e.g. HMP V1V3/V3V5) clustered into OTUs by PSD at 97% sequence similarity need per-read phylogenetic placement on a fixed reference tree to test representative-based annotation.

## When NOT to use

- When the input is already a feature/OTU table without the underlying sequence reads — placement requires individual read sequences
- When the analysis goal is OTU clustering itself rather than auditing representative-based OTU annotation (use PSD/MSD/BLD dissimilarity analyses instead)
- When reads are full-length 16S sequences that can be aligned directly into the reference alignment without fragment-aware placement

## Inputs

- QIIME-generated OTU read collections (HMP V1V3 and V3V5, clustered by PSD at a 97% sequence similarity threshold)
- 16S rRNA reference alignment and reference tree backbone (full-length marker sequences)
- Short amplicon reads in FASTA format ready for phylogenetic placement

## Outputs

- per_read_placements.csv — per-read placement-derived taxonomy from the reference tree
- Discordance table flagging reads whose placement-derived annotation differs from the representative-sequence annotation of their OTU
- Per-OTU summary of the fraction of reads with representative-vs-placement annotation conflicts (e.g. 12.9% V3V5, 19.8% V1V3 OTUs affected)

## How to apply

First construct or obtain a fixed reference tree and alignment for the 16S marker using the reference-based tools indexed in the paper: PASTA for ultra-large multiple sequence alignment, FastTree 2 or RAxML version 8 for approximately-ML phylogeny on large alignments. Next, run SEPP (SATé-enabled phylogenetic placement) to insert each short amplicon read into the reference tree, since short reads cannot be aligned directly to the full-length backbone. Then use pplacer's maximum-likelihood/Bayesian placement to position each sequence onto the fixed reference tree and extract the placed lineage/taxonomic label for every read. Finally, compare each read's placement-derived taxonomy with the annotation it would inherit from its OTU's single representative sequence, flagging discordances — in this HMP analysis, 12.9% of V3V5 and 19.8% of V1V3 OTUs contained such differing reads. The rationale is that because OTUs are typically annotated from one representative sequence, poorly clustered OTUs (where MSD or BLD often exceed the 0.03 threshold even under PSD clustering) can mislabel many member reads.

## Related tools

- **SEPP** (Inserts short 16S amplicon reads into the fixed reference tree (SATé-enabled phylogenetic placement) so fragment reads can be placed on a full-length backbone) — github.com/namphuon/phylo
- **pplacer** (Maximum-likelihood/Bayesian placement engine that positions each sequence onto the fixed reference tree to derive per-read taxonomic labels)
- **RAxML version 8** (Phylogenetic analysis tool in the reference-tree construction lineage for large 16S phylogenies)
- **PASTA** (Ultra-large multiple sequence alignment for nucleotide sequences used to build the 16S reference alignment)
- **FastTree 2** (Approximately maximum-likelihood tree inference for large alignments in the reference-tree construction lineage)
- **QIIME** (Source of the HMP OTU read collections clustered by PSD at 97% sequence similarity that feed into the placement analysis)

## Examples

```
sepp.py -i hmp_v3v5_reads.fasta -r 16s_ref_alignment.fasta -t 16s_reference.tre -a 16s_ref_aligned.fasta -o sepp_v3v5 && python compare_representative_vs_placement.py --placements sepp_v3v5/placement. --otu-map otu_reads.txt --out per_read_placements.csv
```

## Evaluation signals

- Per-read placement table (per_read_placements.csv) contains one placed lineage label for every read in each V1V3/V3V5 OTU read collection
- Discordance rate should be recomputable: 12.9% of V3V5 OTUs and 19.8% of V1V3 OTUs contain at least one read whose annotation differs from the representative's
- Every placed read maps to a branch/node of the fixed reference tree — no unplaced reads should remain after SEPP/pplacer
- Sanity check: reads within OTUs whose MSD/BLD to the representative exceeds the 0.03 threshold should be enriched for annotation discordances
- Placement output should be a valid pplacer/SEPP placement file (e.g. jplace) from which taxonomy strings can be extracted deterministically

## Limitations

- Percent sequence similarity can overestimate evolutionary similarity, so even PSD-97% OTUs may be internally heterogeneous — placement discordance reflects this and cannot be fixed by the tree alone
- The 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation, limiting what representative-based annotation can ever guarantee
- Placement on a fixed reference tree depends on the quality/coverage of the reference alignment and tree; poorly represented lineages may receive unreliable placements

## Evidence

- [abstract] OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses: "As OTUs are typically annotated based upon a single representative sequence, poorly clustered OTUs can have significant impact on downstream analyses"
- [abstract] The representative sequence is annotated, and that annotation is applied to all remaining sequences within that OTU: "This representative sequence is annotated, and that annotation is applied to all remaining sequences within that OTU"
- [intro] the 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation: "the 97% 16S rRNA sequence similarity threshold used to delineate species is only a rough approximation"
- [intro] we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold: "we use a subset of the QIIME-based analyses from the Human Microbiome Project14 (HMP), which computed OTUs using PSD based upon a 97% sequence similarity threshold"
- [abstract] V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree: "V. pplacer: linear time maximum-likelihood and Bayesian phylogenetic placement of sequences onto a fixed reference tree"
