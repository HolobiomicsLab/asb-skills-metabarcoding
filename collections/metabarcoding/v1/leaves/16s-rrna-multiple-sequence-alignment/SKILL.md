---
name: 16s-rrna-multiple-sequence-alignment
description: Use when you have fragmentary 16S rRNA amplicon sequences (e.g. HMP V1V3/V3V5
  OTU members) that must be aligned to an existing backbone reference alignment before
  computing alignment-based dissimilarity metrics such as MSD.
when_to_use_negative:
- You need a de novo multiple sequence alignment of full-length sequences rather than
  placement of fragmentary reads into a fixed backbone alignment
- Your downstream metric is percent sequence dissimilarity (PSD) from pairwise alignment
  rather than the MSA-based MSD described in this paper
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SEPP
  role: UPP 2.0 procedure (SEPP-based) for placing fragmentary OTU sequences into
    the fixed PASTA backbone alignment
  repo: github.com/namphuon/phylo
- name: PASTA
  role: Produces the 16S reference alignment that serves as the fixed backbone for
    UPP placement
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:0520eb42b6d4527b46e31e0f664d01b496bfd8a9e0bbb47ec74c82535e92e11e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/namphuon/phylo
  tools:
  - PASTA
  - SEPP
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: validation_error
yes_rate: 0.0
n_claims: 0
---

# 16s-rrna-multiple-sequence-alignment

## Summary

Reconstructs per-OTU multiple sequence alignments for HMP V1V3 and V3V5 OTU reads by phylogenetically placing each OTU's member sequences into a PASTA-generated 16S reference alignment with UPP 2.0 (SEPP), yielding reference-based alignments used to compute MSA-based sequence dissimilarity (MSD) between representative and member sequences.

## When to use

Use when you have fragmentary 16S rRNA amplicon sequences (e.g. HMP V1V3/V3V5 OTU members) that must be aligned to an existing backbone reference alignment before computing alignment-based dissimilarity metrics such as MSD.

## When NOT to use

- You need a de novo multiple sequence alignment of full-length sequences rather than placement of fragmentary reads into a fixed backbone alignment
- Your downstream metric is percent sequence dissimilarity (PSD) from pairwise alignment rather than the MSA-based MSD described in this paper

## Inputs

- FASTA file of OTU member sequences (HMP V1V3 and V3V5 OTU sequences)
- PASTA-generated 16S rRNA reference backbone alignment

## Outputs

- One FASTA-format multiple sequence alignment per OTU, with all member sequences present at the reference alignment length
- Reference-based alignments ready for downstream MSD computation against representative sequences

## How to apply

Load the OTU member sequences (HMP V1V3 and V3V5 OTU sequences) together with the PASTA-generated 16S reference alignment. Decompose each OTU's sequences and construct an insertion enrichment set following the SEPP-based UPP 2.0 procedure for placing fragmentary sequences into a fixed backbone alignment. Run UPP 2.0 to insert each OTU's member sequences phylogenetically into the PASTA reference alignment, then extract the resulting aligned sequences per OTU. Write one FASTA-format multiple sequence alignment per OTU and verify that every OTU yields a complete alignment file with all member sequences present at the reference alignment length. The rationale is that the MSD (0.03 threshold) is computed on the UPP-generated reference-based alignment rather than a de novo per-OTU MSA, avoiding the pairwise-alignment similarity problems the paper documents.

## Related tools

- **SEPP** (UPP 2.0 procedure (SEPP-based) for placing fragmentary OTU sequences into the fixed PASTA backbone alignment) — github.com/namphuon/phylo
- **PASTA** (Produces the 16S reference alignment that serves as the fixed backbone for UPP placement)

## Evaluation signals

- Every OTU yields a complete alignment file containing all member sequences
- Aligned sequences match the reference alignment length (no sequences dropped or truncated by placement)
- One FASTA-format MSA is produced per OTU, one file for each of the HMP V1V3 and V3V5 OTUs
- MSD values computed on the UPP-generated alignment reproduce the paper's expectation that MSD often exceeds the 0.03 threshold (e.g. 80.5% of V3V5 OTUs had a sequence with MSD>0.03 from the representative)

## Limitations

- The alignment is reference-based: placement quality depends on the PASTA backbone alignment and the SEPP decomposition/enrichment set, not on a per-OTU de novo MSA
- HMP amplicon reads are fragmentary (V1V3/V3V5 hypervariable regions), so placement may be uncertain for sequences poorly covered by the backbone

## Evidence

- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [results] 80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence: "80.5% of the V3V5 OTUs had at least one sequence that was >3% dissimilar (MSD>0.03) from the representative sequence"
- [abstract] PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences: "PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences"
