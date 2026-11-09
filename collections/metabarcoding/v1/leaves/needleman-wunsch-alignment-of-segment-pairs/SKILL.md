---
name: needleman-wunsch-alignment-of-segment-pairs
description: Use when classifying rRNA amplicon or shotgun reads against a full-length
  reference database and you need refined pairwise alignments of k-mer seed hits to
  drive taxonomy and OTU assignments with confidence scores.
when_to_use_negative:
- Input reads are not rRNA (e.g. protein-coding shotgun reads) — MAPseq is designed
  for reference-based rRNA sequence analysis
- You only need approximate read counts and have no need for refined pairwise alignment
  or confidence estimates (the full NW stage is unnecessary overhead)
- Your reference set lacks per-level identity-cutoff annotations, since the combined
  score/identity-cutoff confidence calculation depends on preoptimized cutoffs
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: MAPseq
  role: Reference implementation performing the k-mer search, Needleman-Wunsch alignment
    of HSPs, and confidence estimation in a single run
  repo: https://github.com/jfmrod/mapseq
- name: gcc
  role: C compiler used to build MAPseq from source via ./configure CC="gcc ..."
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:e7576d48b72e3ccec185b78650136458e40e1f70d3febe12235882d888a60223
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - gcc
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# needleman-wunsch-alignment-of-segment-pairs

## Summary

Implements the Needleman-Wunsch alignment stage of MAPseq: high-scoring segment pairs (HSPs) produced by the k-mer search are refined with full pairwise alignment to identify the best reference rRNA match, which feeds the per-read taxonomy/OTU confidence estimates.

## When to use

Use when classifying rRNA amplicon or shotgun reads against a full-length reference database and you need refined pairwise alignments of k-mer seed hits to drive taxonomy and OTU assignments with confidence scores.

## When NOT to use

- Input reads are not rRNA (e.g. protein-coding shotgun reads) — MAPseq is designed for reference-based rRNA sequence analysis
- You only need approximate read counts and have no need for refined pairwise alignment or confidence estimates (the full NW stage is unnecessary overhead)
- Your reference set lacks per-level identity-cutoff annotations, since the combined score/identity-cutoff confidence calculation depends on preoptimized cutoffs

## Inputs

- High-scoring segment pairs / candidate k-mer matches against a full-length rRNA reference FASTA
- FASTA file of demultiplexed, quality-filtered rRNA reads (amplicon or shotgun)
- Reference taxonomy file(s) with per-level identity cutoffs (e.g. '#cutoff: 0.00:0.08 0.70:0.35 ...')

## Outputs

- Per-read alignment records (query id, reference id, alignment score, pairwise identity, matches/mismatches/gaps, start/end positions, strand)
- Per-read taxonomy and OTU assignments with confidence values in [0,1] at each level (MSEQ 'confidences' or 'simple' format)
- per_read_confidence_scores.tsv mapping table (per the reference workflow)

## How to apply

First run the k-mer search stage (k-mer counting with pre-clustering of matches against the reference rRNA database) to generate candidate high-scoring segment pairs. Then perform full Needleman-Wunsch pairwise alignment of each HSP against candidate reference sequences to obtain the alignment score, pairwise identity, matches, mismatches, gaps, and query/reference start-end coordinates reported in the MSEQ output. Use the highest alignment score to select the reference hit, and pass the alignment evidence into the confidence-estimation stage, which combines a score confidence (identity of the assigned taxonomy vs. the first sequence not matching it) with an identity-cutoff confidence (preoptimized per-level cutoffs). Apply the recommended combined confidence cutoff of 0.4 or 0.5, which yielded the highest F1/2-score in the paper's benchmarks, to accept or reject per-level assignments.

## Related tools

- **MAPseq** (Reference implementation performing the k-mer search, Needleman-Wunsch alignment of HSPs, and confidence estimation in a single run) — https://github.com/jfmrod/mapseq
- **gcc** (C compiler used to build MAPseq from source via ./configure CC="gcc ...")

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- Output file exists with per-read rows containing taxonomy assignment and confidence fields
- All confidence values are bounded in [0,1]
- The alignment fields (score, pairwise identity, matches, mismatches, gaps, start/end positions, strand) are populated per read in the MSEQ output
- Implementation runs end-to-end as a self-contained reference (script_runs check) on a small amplicon-style rRNA query set
- Combined confidence at or above the recommended 0.4/0.5 cutoff yields assignments consistent with the simple-format output

## Limitations

- Second hits with missing taxonomic labels (uncertain annotation) can decrease the confidence of the top-hit assignment unless -ignoreEmptyTax is enabled (added in v1.2.4, default off)
- Long queries (>1200 bp) previously caused double hits to be reported (fixed in v1.2.3)
- Low-quality reference sequences caused issues when classifying low-quality query sequences (addressed in mapref-2.2b)
- Moving the mapseq binary alone breaks installation because data files are searched relative to the binary's path

## Evidence

- [abstract] Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] MAPseq is a set of fast and accurate sequence read classification tools designed to assign taxonomy and OTU classifications to ribosomal RNA sequences.: [evidence span withheld — anchor status: external_doc]
- [readme] In the results output, each line indicates a classification of the read. Two output formats can be chosen ("simple" or "confidences") using the --outfmt option.: [evidence span withheld — anchor status: external_doc]
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
