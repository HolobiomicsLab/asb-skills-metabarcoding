---
name: large-scale-sequence-dataset-curation
description: Use when a phylogenetic or OTU-based analysis requires a reference multiple
  sequence alignment built from a large set of full-length 16S rRNA sequences, against
  which shorter OTU fragments will later be aligned.
when_to_use_negative:
- Input consists of short 16S amplicon fragments (e.g. V1–V3 or V3–V5) rather than
  full-length sequences — these are placed onto the reference alignment downstream,
  not used to build it
- Input is already a curated multiple sequence alignment; PASTA alignment estimation
  is unnecessary
- The task is OTU clustering by percent sequence similarity (e.g. 97% PSD), which
  does not require building a reference alignment
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
tools:
- name: PASTA
  role: Estimates the reference multiple sequence alignment from the curated RDP full-length
    16S sequences via iterative decomposition, alignment, and maximum-likelihood subtree
    merging (version 1.6.3 in the source workflow)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:eef133b48cb70438e52dbb5cc4e31728bb3d8ba9c4ff0dc90490cadc8817c416
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTA
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.3333333333333333
n_claims: 3
---

# large-scale-sequence-dataset-curation

## Summary

Curate a large reference sequence set (high-quality full-length 16S rRNA sequences from the RDP database) and estimate a single reference multiple sequence alignment with PASTA, providing the alignment substrate for downstream OTU-level metrics such as MSD and BLD.

## When to use

Use when a phylogenetic or OTU-based analysis requires a reference multiple sequence alignment built from a large set of full-length 16S rRNA sequences, against which shorter OTU fragments will later be aligned.

## When NOT to use

- Input consists of short 16S amplicon fragments (e.g. V1–V3 or V3–V5) rather than full-length sequences — these are placed onto the reference alignment downstream, not used to build it
- Input is already a curated multiple sequence alignment; PASTA alignment estimation is unnecessary
- The task is OTU clustering by percent sequence similarity (e.g. 97% PSD), which does not require building a reference alignment

## Inputs

- High-quality full-length 16S rRNA sequences downloaded from the RDP database
- Curated FASTA sequence set (full-length, high-quality 16S sequences retained after curation)

## Outputs

- reference_alignment.fasta — a PASTA-estimated reference multiple sequence alignment in FASTA format with one aligned row per input sequence and a common alignment length across all rows

## How to apply

Start by downloading the high-quality full-length 16S rRNA sequences from the RDP database, then curate the set to retain only full-length, high-quality sequences suitable for alignment. Run PASTA (version 1.6.3 in the source workflow), which iterates decomposition, alignment, and maximum-likelihood subtree merging, on the curated RDP sequence set to estimate a single reference multiple sequence alignment. Write the PASTA-estimated alignment to reference_alignment.fasta in FASTA format, keeping one aligned row per input sequence. The rationale is that pairwise-alignment-based similarity can yield poorly clustered OTUs, so downstream dissimilarity metrics (MSD, BLD) are computed on a robust large-scale reference alignment instead.

## Related tools

- **PASTA** (Estimates the reference multiple sequence alignment from the curated RDP full-length 16S sequences via iterative decomposition, alignment, and maximum-likelihood subtree merging (version 1.6.3 in the source workflow))

## Examples

```
python run_pasta.py -i curated_rdp_16S.fasta --outdir pasta_ref --alignment-method pastajs
```

## Evaluation signals

- reference_alignment.fasta exists and is parseable as a FASTA alignment
- The number of aligned sequences in reference_alignment.fasta equals the number of curated RDP input sequences
- All rows in the alignment share a common alignment length (one aligned row per input sequence)
- The PASTA run used a pinned version (1.6.3) so the alignment is reproducible
- Only full-length, high-quality 16S sequences remain in the curated input set before alignment

## Limitations

- PASTA is designed for ultra-large alignments; for small curated sets a simpler aligner may suffice
- Quality filtering of the RDP full-length 16S set is a prerequisite — including low-quality or partial sequences degrades the reference alignment
- Pairwise sequence alignment-based similarity computation can result in poorly clustered OTUs, which motivates but also constrains how the reference alignment may be reused

## Evidence

- [methods] A reference multiple sequence alignment was estimated using PASTA version 1.6.3 on the high-quality full-length 16S sequences from the RDP database: [evidence span withheld — anchor status: not_found]
- [methods] verify that reference_alignment.fasta exists, is parseable as a FASTA alignment, and that the number of aligned sequences equals the number of curated RDP input sequences: [evidence span withheld — anchor status: not_found]
- [abstract] PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences: "PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences"
