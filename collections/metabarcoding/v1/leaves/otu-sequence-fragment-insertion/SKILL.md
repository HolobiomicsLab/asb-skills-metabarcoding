---
name: otu-sequence-fragment-insertion
description: Use when OTU member sequences are short amplicon fragments (e.g., HMP
  V1V3 or V3V5 reads) that must be aligned to a fixed reference alignment before computing
  MSD between representative and member sequences.
when_to_use_negative:
- Sequences are full-length and a de novo multiple sequence alignment per OTU is feasible/required
- The analysis goal is percent sequence dissimilarity (PSD) from pairwise alignments
  rather than MSA-based dissimilarity (MSD)
- No pre-existing reference alignment backbone is available and one has not been generated
  (e.g., via PASTA)
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0084
- http://edamontology.org/topic_3697
tools:
- name: SEPP
  role: SATé-enabled phylogenetic placement engine underlying UPP 2.0; places fragmentary
    OTU sequences into the fixed reference alignment via insertion enrichment sets
  repo: github.com/namphuon/phylo
- name: PASTA
  role: Produces the ultra-large 16S rRNA multiple sequence alignment that serves
    as the fixed reference backbone into which OTU reads are inserted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:70ba724eadda985c1d140d12392cbe86bbbdf30e59798cc5cfc764a438819038
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTA
  - SEPP
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0084
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# otu-sequence-fragment-insertion

## Summary

Places fragmentary 16S rRNA OTU member sequences (HMP V1V3 and V3V5 amplicon reads) phylogenetically into an existing PASTA-generated 16S reference alignment using SEPP-based UPP 2.0, yielding one reference-aligned FASTA MSA per OTU for downstream MSA-based sequence dissimilarity (MSD) computation.

## When to use

Use when OTU member sequences are short amplicon fragments (e.g., HMP V1V3 or V3V5 reads) that must be aligned to a fixed reference alignment before computing MSD between representative and member sequences.

## When NOT to use

- Sequences are full-length and a de novo multiple sequence alignment per OTU is feasible/required
- The analysis goal is percent sequence dissimilarity (PSD) from pairwise alignments rather than MSA-based dissimilarity (MSD)
- No pre-existing reference alignment backbone is available and one has not been generated (e.g., via PASTA)

## Inputs

- HMP V1V3 OTU member sequences (FASTA, one set per OTU)
- HMP V3V5 OTU member sequences (FASTA, one set per OTU)
- PASTA-generated 16S rRNA reference alignment (fixed backbone alignment)

## Outputs

- One FASTA-format multiple sequence alignment per OTU, with all member sequences inserted at the reference alignment length
- Reference-based alignments ready for per-OTU MSD computation between representative and member sequences

## How to apply

Load the OTU member sequences (HMP V1V3 and V3V5 OTU reads) together with the PASTA-generated 16S reference alignment. Following the SEPP-based UPP 2.0 procedure, decompose each OTU's sequences and construct an insertion enrichment set so that fragmentary sequences can be placed onto the fixed backbone alignment. Run UPP 2.0 to insert each OTU's member sequences phylogenetically into the PASTA reference alignment, then extract the resulting aligned sequences and write one FASTA-format MSA per OTU. The rationale is that pairwise alignment-based similarity (PSD) can result in poorly clustered OTUs, so MSD must be computed on a reference-based alignment rather than a de novo MSA per OTU. Verify each output alignment is complete before downstream MSD computation.

## Related tools

- **SEPP** (SATé-enabled phylogenetic placement engine underlying UPP 2.0; places fragmentary OTU sequences into the fixed reference alignment via insertion enrichment sets) — github.com/namphuon/phylo
- **PASTA** (Produces the ultra-large 16S rRNA multiple sequence alignment that serves as the fixed reference backbone into which OTU reads are inserted)

## Evaluation signals

- Every OTU yields a complete alignment file with all member sequences present
- Each output sequence has length equal to the PASTA reference alignment length (no unaligned raw fragments remain)
- MSD computed on the UPP-generated reference-based alignment, confirming no de novo per-OTU MSA was substituted
- Both HMP V1V3 and V3V5 OTU sets are covered (no missing OTU alignment files)
- Downstream MSD values are reproducible against reported expectations (e.g., 80.5% of V3V5 OTUs had at least one sequence with MSD>0.03 from the representative)

## Limitations

- Results depend on the quality and taxonomic coverage of the PASTA-generated reference alignment backbone
- Fragmentary amplicon reads (V1V3/V3V5) cannot be reliably aligned de novo; the placement approach inherits SEPP/UPP heuristic decomposition choices
- The 97% similarity threshold used to define the OTUs is only a rough approximation for delineating species, so MSD on placed reads reflects OTU definitions rather than true species boundaries

## Evidence

- [methods] Each OTU's sequences were aligned to the 16S reference alignment using UPP 2.0 (aligning fragmentary sequences to an existing reference): [evidence span withheld — anchor status: not_found]
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [abstract] the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs: "the traditional approach of using pairwise sequence alignments to compute sequence similarity can result in poorly clustered OTUs"
- [abstract] SEPP: SATé-enabled phylogenetic placement: "SEPP: SATé-enabled phylogenetic placement"
