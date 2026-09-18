---
name: 16s-rrna-reference-alignment-construction
description: Use when you need a full-length 16S rRNA reference MSA to serve as the
  substrate for aligning OTU fragment sequences (e.g. HMP V1V3/V3V5) before computing
  MSD or BLD between representative and member sequences.
when_to_use_negative:
- Input sequences are short amplicon fragments (e.g. V3V5 OTU sequences) rather than
  full-length 16S — those are aligned against the already-built reference, not used
  to build it
- You only need pairwise similarity for OTU clustering at a 97% percent-similarity
  threshold; the reference alignment is for MSD/BLD-style downstream computation,
  not for PSD clustering
edam_operation: http://edamontology.org/operation_0492
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: PASTA
  role: Estimates the reference multiple sequence alignment (v1.6.3) from the curated
    RDP full-length 16S sequence set by iterating decomposition, alignment, and maximum-likelihood
    subtree merging
- name: RDP database
  role: Source of the high-quality full-length 16S rRNA reference sequences used as
    PASTA input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/npjbiofilms.2016.4
    title: A perspective on 16S rRNA operational taxonomic unit clustering using sequence
      similarity
  source_content_hash: sha256:857bb82003f7ebc77d6c7325a2847f320fd9eeb7af993e3ca079a0c90fd79d3b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PASTA
  - RDP database
  edam_operation: http://edamontology.org/operation_0492
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# 16s-rrna-reference-alignment-construction

## Summary

Estimates a single reference multiple sequence alignment of high-quality full-length 16S rRNA sequences from the RDP database using PASTA version 1.6.3, producing the alignment substrate on which OTU fragments are aligned and downstream MSD/BLD dissimilarity computations are performed.

## When to use

Use when you need a full-length 16S rRNA reference MSA to serve as the substrate for aligning OTU fragment sequences (e.g. HMP V1V3/V3V5) before computing MSD or BLD between representative and member sequences.

## When NOT to use

- Input sequences are short amplicon fragments (e.g. V3V5 OTU sequences) rather than full-length 16S — those are aligned against the already-built reference, not used to build it
- You only need pairwise similarity for OTU clustering at a 97% percent-similarity threshold; the reference alignment is for MSD/BLD-style downstream computation, not for PSD clustering

## Inputs

- High-quality full-length 16S rRNA sequences from the RDP database (FASTA)
- Curated full-length, high-quality 16S sequence set suitable for alignment

## Outputs

- reference_alignment.fasta: PASTA-estimated reference multiple sequence alignment in FASTA format, one aligned row per curated input sequence with a common alignment length

## How to apply

Download the high-quality full-length 16S rRNA sequences from the RDP database and curate the set, retaining only full-length, high-quality sequences suitable for alignment. Run PASTA (version 1.6.3 in this study), which iterates decomposition, alignment, and maximum-likelihood subtree merging, on the curated RDP sequence set to estimate a single reference multiple sequence alignment. Write the PASTA-estimated alignment to reference_alignment.fasta in FASTA format, keeping one aligned row per input sequence. This reference alignment is the substrate for subsequent OTU fragment alignment and for the MSD and BLD computations between each OTU representative sequence and its member sequences.

## Related tools

- **PASTA** (Estimates the reference multiple sequence alignment (v1.6.3) from the curated RDP full-length 16S sequence set by iterating decomposition, alignment, and maximum-likelihood subtree merging)
- **RDP database** (Source of the high-quality full-length 16S rRNA reference sequences used as PASTA input)

## Examples

```
python run_pasta.py -i curated_rdp_full_length_16s.fasta --output reference_alignment.fasta
```

## Evaluation signals

- reference_alignment.fasta exists and is parseable as a FASTA alignment
- The number of aligned sequences equals the number of curated RDP input sequences
- All rows in reference_alignment.fasta share a common alignment length
- The alignment retains one aligned row per input sequence (no dropped rows)
- The alignment supports downstream OTU fragment alignment followed by MSD and BLD computation between representative and member sequences

## Limitations

- The quality of downstream MSD/BLD dissimilarity values depends on the accuracy of the PASTA-estimated reference alignment
- Only full-length, high-quality 16S sequences should enter the PASTA run; poor-quality or partial input degrades the reference alignment
- The paper's broader finding that pairwise similarity clustering (PSD at 97%) yields poorly clustered OTUs motivates but also contextualizes this alignment-based approach — the reference alignment alone does not fix OTU delineation problems

## Evidence

- [methods] A reference multiple sequence alignment was estimated using PASTA version 1.6.3 on the high-quality full-length 16S sequences from the RDP database: [evidence span withheld — anchor status: not_found]
- [intro] We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD: "We take the V1V3 and V3V5 OTUs that were generated from the HMP and compute the MSD and BLD between the representative sequence and all remaining sequences within the OTU"
- [abstract] PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences: "PASTA: ultra-large multiple sequence alignment for nucleotide and amino-acid sequences"
