---
name: reference-implementation-validation
description: Use when a task asks to reproduce or validate an algorithmic pipeline
  described in a paper's methods/supplementary material from its source repository,
  producing per-read confidence-scored assignments as a checkable artifact.
when_to_use_negative:
- The input is a denoised/ASV feature table or pre-computed abundance matrix rather
  than raw rRNA read sequences
- The goal is de novo OTU clustering or reference-free classification, since MAPseq
  is explicitly a reference-based rRNA analysis framework
- Only a summary count table is needed and running a full per-read reference implementation
  would be unnecessary
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: MAPseq
  role: Reference implementation of the k-mer search, Needleman-Wunsch HSP alignment
    and confidence-estimation stages used for validation
  repo: https://github.com/jfmrod/mapseq
- name: gcc
  role: C compiler used to build the reference implementation from source via ./configure
    and make
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:c856d7014528b02bc91fd086b1cef4dbb56a7c4970fe7b2a5e3f2cf9fec6197a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - gcc
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.2857142857142857
n_claims: 7
---

# reference-implementation-validation

## Summary

Reconstructs MAPseq's three algorithmic stages — improved k-mer counting with a pre-clustering step, full Needleman-Wunsch alignment of high scoring segment pairs, and the classification-confidence algorithm — as a self-contained reference implementation, then validates it end-to-end on a small rRNA query set against the published behavior.

## When to use

Use when a task asks to reproduce or validate an algorithmic pipeline described in a paper's methods/supplementary material from its source repository, producing per-read confidence-scored assignments as a checkable artifact.

## When NOT to use

- The input is a denoised/ASV feature table or pre-computed abundance matrix rather than raw rRNA read sequences
- The goal is de novo OTU clustering or reference-free classification, since MAPseq is explicitly a reference-based rRNA analysis framework
- Only a summary count table is needed and running a full per-read reference implementation would be unnecessary

## Inputs

- MAPseq source repository (https://github.com/jfmrod/mapseq) with in-repository methods documentation
- Small rRNA query set in FASTA format (amplicon-style reads, demultiplexed and quality filtered)
- Reference rRNA sequence FASTA with tab-separated taxonomy file(s) (e.g. customref.fasta, customref.tax with #cutoff/#name/#levels header)

## Outputs

- per_read_confidence_scores.tsv with per-read rows containing taxonomy assignment and confidence fields bounded in [0,1]
- Per-read mapping of query reads to reference sequences with alignment score, pairwise identity, matches/mismatches/gaps, and per-level taxonomy confidences

## How to apply

Retrieve the MAPseq source from https://github.com/jfmrod/mapseq and extract the descriptions of the three stages (k-mer counting with pre-clustering, HSP alignment, confidence estimation) from the repository's methods documentation. Implement the k-mer counting stage with pre-clustering of k-mer matches against the reference rRNA database for candidate generation, then apply full Needleman-Wunsch pairwise alignment to the high scoring segment pairs to refine candidate reference matches. Convert alignment evidence into per-read confidence scores using the combined confidence scheme: a score confidence (identity of the assigned taxonomy vs. the first sequence not matching it) and an identity-cutoff confidence using preoptimized per-level cutoffs; a combined cutoff of 0.4–0.5 was recommended by the authors. Build the reference implementation with the autotools chain (./bootstrap, ./configure, make, make install, with gcc) and run it on a small amplicon-style rRNA FASTA query set. Write per-read mapping results with taxonomy assignment and confidence fields to per_read_confidence_scores.tsv and verify end-to-end execution.

## Related tools

- **MAPseq** (Reference implementation of the k-mer search, Needleman-Wunsch HSP alignment and confidence-estimation stages used for validation) — https://github.com/jfmrod/mapseq
- **gcc** (C compiler used to build the reference implementation from source via ./configure and make)

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- per_read_confidence_scores.tsv exists and every row contains a taxonomy assignment plus confidence fields
- All confidence values are bounded in [0,1]
- The implementation runs end-to-end as a self-contained reference (script_runs check) without errors
- The three implemented stages reproduce the pipeline described for MAPseq's reference-based rRNA analysis (k-mer pre-clustering, NW alignment of HSPs, confidence estimation)
- Output format matches the documented mseq 'confidences' schema: query id, reference id, alignment score, pairwise identity, matches, mismatches, gaps, alignment coordinates, strand, then per-level taxonomy confidences

## Limitations

- Binary packages place data files relative to the binary's path — moving only the binary breaks the installation, so full source builds must preserve the directory layout
- Dropped or missing taxonomic labels (uncertain annotation) can lower confidence of the top-hit assignment; the -ignoreEmptyTax option exists but was default-off pending benchmarks
- Very low-quality reference sequences can cause issues when classifying low-quality query sequences (fixed in mapref-2.2b); versions before 1.2.2 had a multithreaded race condition

## Evidence

- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster"
- [readme] score confidence calculation: [evidence span withheld — anchor status: external_doc]
- [readme] confidence cutoff recommendation: [evidence span withheld — anchor status: external_doc]
- [methods] build procedure: "cd' to the directory containing the package's source code and type './configure' to configure the package for your system."
- [readme] basic classification run: [evidence span withheld — anchor status: external_doc]
