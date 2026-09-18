---
name: fasta-alignment-file-handling
description: Use when OTU member sequences must be extracted into per-OTU FASTA alignment
  files of reference-alignment length, or when a reference FASTA alignment must be
  validated before computing MSD/BLD.
when_to_use_negative:
- When a de novo per-OTU MSA is the goal — this skill produces reference-based alignments
  via UPP insertion, not independent per-OTU multiple sequence alignments
- When sequences are already aligned and only dissimilarity metrics (MSD/BLD) need
  computing — file handling is a prerequisite step, not the analysis itself
- When inputs are unassembled reads lacking OTU membership — clustering into OTUs
  must precede per-OTU alignment extraction
edam_operation: http://edamontology.org/operation_0491
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: SEPP
  role: UPP 2.0 (SEPP-based) decomposes OTU sequences, builds an insertion enrichment
    set, and inserts fragmentary OTU reads into the fixed PASTA backbone alignment
- name: PASTA
  role: Estimates the version 1.6.3 reference multiple sequence alignment of RDP full-length
    16S sequences that serves as the alignment substrate
merged_aliases:
- nucleotide-alignment-file-handling
merged_alias_records:
- alias: nucleotide-alignment-file-handling
  slug: nucleotide-alignment-file-handling
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:c9426e3a376ed12ecbae8361f1bd6975f7c866030567bf526e987b5559e8184f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTA
  - SEPP
  edam_operation: http://edamontology.org/operation_0491
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.25
n_claims: 4
---

# fasta-alignment-file-handling

## Summary

Writing and validating FASTA-format multiple sequence alignments in the 16S rRNA MSD/BLD pipeline: one per-OTU MSA file produced by UPP 2.0 insertion into a PASTA reference alignment, with structural checks (parseability, sequence count, common alignment length) before downstream dissimilarity computation.

## When to use

Use when OTU member sequences must be extracted into per-OTU FASTA alignment files of reference-alignment length, or when a reference FASTA alignment must be validated before computing MSD/BLD.

## When NOT to use

- When a de novo per-OTU MSA is the goal — this skill produces reference-based alignments via UPP insertion, not independent per-OTU multiple sequence alignments
- When sequences are already aligned and only dissimilarity metrics (MSD/BLD) need computing — file handling is a prerequisite step, not the analysis itself
- When inputs are unassembled reads lacking OTU membership — clustering into OTUs must precede per-OTU alignment extraction

## Inputs

- PASTA-generated reference multiple sequence alignment of curated RDP full-length 16S sequences (reference_alignment.fasta)
- HMP V1V3 OTU member sequences (unaligned fragmentary reads)
- HMP V3V5 OTU member sequences (unaligned fragmentary reads)
- OTU membership/representative-sequence definitions from the 97% PSD clustering

## Outputs

- One FASTA-format multiple sequence alignment per OTU, with all member sequences at the reference alignment length
- Validated reference_alignment.fasta (parseable, one row per input RDP sequence, common alignment length)

## How to apply

First obtain the PASTA version 1.6.3 reference alignment of curated full-length RDP 16S sequences and write it to reference_alignment.fasta, confirming it parses as FASTA with all input rows at a single common alignment length. Then load the HMP V1V3 and V3V5 OTU member sequences and run UPP 2.0 (the SEPP-based procedure for placing fragmentary sequences into a fixed backbone) to insert each OTU's members phylogenetically into the PASTA reference alignment. Extract the aligned sequences for each OTU and write one FASTA-format multiple sequence alignment per OTU. Finally, verify each OTU file exists, is parseable, contains every member sequence, and that all rows match the reference alignment length so MSD between representative and member sequences can be computed row-wise on the same coordinate system.

## Related tools

- **SEPP** (UPP 2.0 (SEPP-based) decomposes OTU sequences, builds an insertion enrichment set, and inserts fragmentary OTU reads into the fixed PASTA backbone alignment)
- **PASTA** (Estimates the version 1.6.3 reference multiple sequence alignment of RDP full-length 16S sequences that serves as the alignment substrate)

## Evaluation signals

- reference_alignment.fasta parses as FASTA and the number of aligned rows equals the number of curated RDP input sequences
- All rows in reference_alignment.fasta share a single common alignment length
- Every OTU yields a complete alignment file with all member sequences present
- Each per-OTU alignment's row length matches the reference alignment length exactly (no ragged rows)
- The per-OTU alignment is the UPP-inserted, reference-based alignment on which MSD>0.03 dissimilarity is then computed against the representative sequence

## Limitations

- Alignments are reference-based (UPP insertion into a fixed PASTA backbone), so alignment quality depends on the reference alignment and insertion procedure rather than per-OTU de novo alignment
- MSD computed on these alignments exceeded the 0.03 threshold for 80.5% of V3V5 OTUs (at least one sequence), reflecting that percent-similarity OTUs are not compact under MSA-based dissimilarity
- The repository README for namphuon/phylo provides no additional usage documentation, so file-handling specifics must be reconstructed from the workflow description

## Evidence

- [other] Extract the resulting aligned sequences for each OTU and write one FASTA-format multiple sequence alignment per OTU.: [evidence span withheld — anchor status: not_found]
- [other] verify that reference_alignment.fasta exists, is parseable as a FASTA alignment, and that the number of aligned sequences equals the number of curated RDP input sequences: [evidence span withheld — anchor status: not_found]
- [other] Run UPP 2.0 to insert each OTU's member sequences phylogenetically into the PASTA reference alignment.: [evidence span withheld — anchor status: not_found]
- [other] A reference multiple sequence alignment was estimated using PASTA version 1.6.3 on the high-quality full-length 16S sequences from the RDP database: [evidence span withheld — anchor status: not_found]
