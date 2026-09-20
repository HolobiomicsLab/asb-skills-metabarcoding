---
name: multiple-sequence-alignment-estimation
description: Use when you need a reference alignment of a large full-length 16S rRNA
  set (e.g., RDP sequences) to serve as the substrate for aligning OTU fragments and
  computing MSD or BLD between representative and member sequences.
when_to_use_negative:
- Input sequences are short 16S amplicon fragments (e.g., V1–V3 or V3–V5 regions)
  rather than full-length 16S — these are aligned against an existing reference alignment,
  not used to build one.
- The analysis only requires percent sequence similarity (PSD) OTU clustering at 97%
  and does not need phylogenetically-aware MSD/BLD distances from an alignment.
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0157
- http://edamontology.org/topic_0092
tools:
- name: PASTA
  role: Estimates the reference multiple sequence alignment of curated RDP full-length
    16S sequences via iterative decomposition, alignment, and maximum-likelihood subtree
    merging (version 1.6.3 used)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:97217be46bccb98fe3dea5ec3f3f2f8787048aaae09b09444833f55ae3ebbc3e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTA
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0157
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# multiple-sequence-alignment-estimation

## Summary

Estimate a single reference multiple sequence alignment of high-quality full-length 16S rRNA sequences (here, from the RDP database) using PASTA, so that downstream OTU fragments can be aligned and MSD/BLD dissimilarities computed against a consistent alignment substrate.

## When to use

Use when you need a reference alignment of a large full-length 16S rRNA set (e.g., RDP sequences) to serve as the substrate for aligning OTU fragments and computing MSD or BLD between representative and member sequences.

## When NOT to use

- Input sequences are short 16S amplicon fragments (e.g., V1–V3 or V3–V5 regions) rather than full-length 16S — these are aligned against an existing reference alignment, not used to build one.
- The analysis only requires percent sequence similarity (PSD) OTU clustering at 97% and does not need phylogenetically-aware MSD/BLD distances from an alignment.

## Inputs

- Curated set of high-quality full-length 16S rRNA sequences from the RDP database (FASTA)
- PASTA version 1.6.3 executable

## Outputs

- reference_alignment.fasta — a reference multiple sequence alignment in FASTA format, one aligned row per input sequence, all rows sharing a common alignment length

## How to apply

Download high-quality full-length 16S rRNA sequences from the RDP database and curate the set, retaining only full-length, high-quality sequences suitable for alignment. Run PASTA version 1.6.3, which iterates decomposition, alignment, and maximum-likelihood subtree merging, to estimate a single reference multiple sequence alignment of the curated set. Write the estimated alignment to reference_alignment.fasta in FASTA format, keeping one aligned row per input sequence. Validate that the FASTA is parseable, that the number of aligned sequences equals the number of curated RDP input sequences, and that all rows share a common alignment length. This alignment is the substrate for subsequent OTU fragment alignment and downstream MSD/BLD computation on HMP V1V3/V3V5 OTU sequences.

## Related tools

- **PASTA** (Estimates the reference multiple sequence alignment of curated RDP full-length 16S sequences via iterative decomposition, alignment, and maximum-likelihood subtree merging (version 1.6.3 used))

## Examples

```
python run_pasta.py -i rdp_full_length_16S.fasta -o pasta_run
```

## Evaluation signals

- reference_alignment.fasta exists and is parseable as a FASTA alignment
- Number of aligned sequences equals the number of curated RDP input sequences
- All aligned rows share a common alignment length
- Alignment was produced with PASTA version 1.6.3 on the curated full-length, high-quality RDP subset
- OTU fragments can subsequently be aligned to the reference and yield valid MSD/BLD computations

## Limitations

- Requires full-length, high-quality 16S input sequences; fragmentary or low-quality RDP sequences must be removed before alignment
- PASTA is designed for ultra-large nucleotide and amino-acid alignments; smaller datasets may be handled adequately by simpler aligners

## Evidence

- [abstract] PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences: "PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences"
- [other] A reference multiple sequence alignment was estimated using PASTA version 1.6.3 on the high-quality full-length 16S sequences from the RDP database: [evidence span withheld — anchor status: not_found]
- [other] Run PASTA (which iterates decomposition, alignment, and maximum-likelihood subtree merging) on the curated RDP sequence set: [evidence span withheld — anchor status: not_found]
- [other] verify that reference_alignment.fasta exists, is parseable as a FASTA alignment, and that the number of aligned sequences equals the number of curated RDP input sequences: [evidence span withheld — anchor status: not_found]
