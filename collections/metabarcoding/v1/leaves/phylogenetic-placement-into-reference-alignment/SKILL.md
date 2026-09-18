---
name: phylogenetic-placement-into-reference-alignment
description: Use when you have fragmentary OTU member sequences (e.g., HMP V1V3/V3V5
  amplicons) that must be aligned to an existing reference alignment to compute an
  alignment-based dissimilarity metric such as MSD.
when_to_use_negative:
- When you need a de novo multiple sequence alignment per OTU rather than placement
  into a fixed reference alignment — this skill computes MSD on the UPP-generated
  reference-based alignment, not a per-OTU de novo MSA.
- When the goal is percent sequence similarity (PSD) clustering of OTUs; this skill
  is for alignment-based (MSD/BLD) assessment of already-clustered OTUs.
- When sequences are full-length and can be aligned directly without phylogenetic
  placement of fragmentary reads into a backbone.
edam_topics:
- http://edamontology.org/topic_0084
tools:
- name: SEPP
  role: SATé-enabled phylogenetic placement engine underlying UPP 2.0; decomposes
    OTU sequences and constructs the insertion enrichment set for placing fragmentary
    sequences into the fixed backbone alignment
  repo: github.com/namphuon/phylo
- name: PASTA
  role: Provides the ultra-large 16S rRNA reference alignment (fixed backbone) into
    which OTU member sequences are inserted by UPP 2.0
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:111614ea7eb90525b41237a36e5dc2d47a823cfc873570b86020c078934e33be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - PASTA
  - SEPP
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 6
---

# phylogenetic-placement-into-reference-alignment

## Summary

Place fragmentary 16S rRNA amplicon sequences (HMP V1V3 and V3V5 OTU reads) phylogenetically into a fixed PASTA-generated reference alignment using SEPP-based UPP 2.0, yielding one reference-aligned FASTA MSA per OTU that serves as the basis for computing MSA-based sequence dissimilarity (MSD) between representative and member sequences.

## When to use

Use when you have fragmentary OTU member sequences (e.g., HMP V1V3/V3V5 amplicons) that must be aligned to an existing reference alignment to compute an alignment-based dissimilarity metric such as MSD.

## When NOT to use

- When you need a de novo multiple sequence alignment per OTU rather than placement into a fixed reference alignment — this skill computes MSD on the UPP-generated reference-based alignment, not a per-OTU de novo MSA.
- When the goal is percent sequence similarity (PSD) clustering of OTUs; this skill is for alignment-based (MSD/BLD) assessment of already-clustered OTUs.
- When sequences are full-length and can be aligned directly without phylogenetic placement of fragmentary reads into a backbone.

## Inputs

- OTU member sequences from HMP V1V3 and V3V5 regions (fragmentary 16S rRNA amplicon sequences, FASTA)
- PASTA-generated 16S rRNA reference alignment (fixed backbone alignment)

## Outputs

- One FASTA-format multiple sequence alignment per OTU, with all member sequences placed at the reference alignment length
- UPP-generated reference-based alignments ready for downstream MSD computation between representative and member sequences

## How to apply

Load the OTU member sequences (HMP V1V3 and V3V5 OTU sequences) together with the PASTA-generated 16S reference alignment, then follow the SEPP-based UPP 2.0 procedure for placing sequences into a fixed backbone alignment: decompose each OTU's sequences and construct an insertion enrichment set. Run UPP 2.0 to insert each OTU's member sequences phylogenetically into the PASTA reference alignment, rather than building a de novo MSA per OTU. Extract the resulting aligned sequences and write one FASTA-format multiple sequence alignment per OTU. The rationale is that pairwise alignment-based similarity computation can result in poorly clustered OTUs, so MSD is computed on the UPP-generated reference-based alignment instead. Verify that every OTU yields a complete alignment file with all member sequences present at the reference alignment length before downstream MSD computation.

## Related tools

- **SEPP** (SATé-enabled phylogenetic placement engine underlying UPP 2.0; decomposes OTU sequences and constructs the insertion enrichment set for placing fragmentary sequences into the fixed backbone alignment) — github.com/namphuon/phylo
- **PASTA** (Provides the ultra-large 16S rRNA reference alignment (fixed backbone) into which OTU member sequences are inserted by UPP 2.0)

## Evaluation signals

- Every OTU yields a complete alignment file after UPP placement.
- All member sequences are present in each per-OTU alignment (no member dropped during placement).
- Every sequence in a per-OTU alignment has length equal to the PASTA reference alignment length.
- MSD computed on the UPP-generated alignment reproduces the paper-style distributions (e.g., ~80.5% of V3V5 OTUs having at least one sequence with MSD>0.03 from the representative; ~2.7% of all V3V5 sequences with MSD>3%).
- Alignments are reference-based (columns correspond to the PASTA backbone) rather than de novo per-OTU MSAs.

## Limitations

- MSD quality depends on the accuracy of the phylogenetic placement of fragmentary reads; misplacement into the backbone corrupts the alignment and hence the MSD.
- The approach addresses the known shortcoming that pairwise alignment-based similarity computation can result in poorly clustered OTUs; it does not itself re-cluster OTUs or fix annotation based on a single representative sequence.
- The 97% sequence similarity threshold used to delineate species is only a rough approximation, so downstream interpretation of OTU compactness must account for that.

## Evidence

- [abstract] SEPP: SATé-enabled phylogenetic placement: "SEPP: SATé-enabled phylogenetic placement"
- [abstract] PASTA ultra-large multiple sequence alignment: "PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences"
- [abstract] Pairwise alignment shortcoming: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
- [methods] UPP 2.0 reference-based alignment workflow: [evidence span withheld — anchor status: not_found]
- [methods] Run UPP 2.0 insertion step: [evidence span withheld — anchor status: not_found]
- [results] MSD results on V3V5 OTUs: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
