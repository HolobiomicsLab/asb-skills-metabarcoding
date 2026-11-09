---
name: rrna-kmer-counting-with-preclustering
description: Use when classifying amplicon or shotgun rRNA reads against a full-length
  reference database with per-read taxonomy/OTU assignments and confidence scores,
  or when reconstructing the k-mer search, NW alignment, and confidence-estimation
  stages of MAPseq.
when_to_use_negative:
- Input reads are not demultiplexed and quality-filtered rRNA sequences (MAPseq expects
  previously filtered FASTA input)
- You need de novo OTU clustering without any reference database — MAPseq is explicitly
  a reference-based rRNA analysis framework
- You only need alignment scoring without taxonomy/OTU assignment and confidence estimation
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3174
tools:
- name: MAPseq
  role: Reference implementation of the k-mer search with pre-clustering, Needleman-Wunsch
    HSP alignment and confidence estimation for rRNA classification
  repo: https://github.com/jfmrod/mapseq
- name: gcc
  role: Compiler used to build MAPseq from source via ./configure CC="gcc ..." and
    make
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:a7da1b7d27697cf4cfaf54e2abfb9c9a09a87183af9cff20f762d8920a6d0fbf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - gcc
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# rrna-kmer-counting-with-preclustering

## Summary

Reimplements MAPseq's k-mer search stage: pre-clustered k-mer counting against a full-length rRNA reference database to generate candidate matches, followed by Needleman-Wunsch alignment of high-scoring segment pairs and computation of per-read classification confidence scores. This is the algorithmic basis for MAPseq being up to one hundred times faster and up to 30% more accurate (F½ score) than existing rRNA classifiers.

## When to use

Use when classifying amplicon or shotgun rRNA reads against a full-length reference database with per-read taxonomy/OTU assignments and confidence scores, or when reconstructing the k-mer search, NW alignment, and confidence-estimation stages of MAPseq.

## When NOT to use

- Input reads are not demultiplexed and quality-filtered rRNA sequences (MAPseq expects previously filtered FASTA input)
- You need de novo OTU clustering without any reference database — MAPseq is explicitly a reference-based rRNA analysis framework
- You only need alignment scoring without taxonomy/OTU assignment and confidence estimation

## Inputs

- FASTA file of demultiplexed, quality-filtered rRNA reads (e.g. rawseqs.fa from FASTQ)
- Nucleotide FASTA reference set of full-length rRNA sequences (e.g. customref.fasta)
- One or more tab-separated taxonomy assignment files (e.g. customref.tax) with a # header declaring identity cutoffs, taxonomy name and levels

## Outputs

- MSEQ output file (e.g. rawseqs.fa.mseq) with per-read alignment fields (query id, reference id, alignment score, pairwise identity, matches, mismatches, gaps, start/end positions, strand) plus taxonomy and OTU classifications with confidences
- Per-read confidence score table (per_read_confidence_scores.tsv) with taxonomy assignment and confidence fields in [0,1]
- Optional OTU/taxa count table via -otucounts or -otutable summary modes

## How to apply

Retrieve the MAPseq source from https://github.com/jfmrod/mapseq and implement three stages: (1) improved k-mer counting based on a pre-clustering step of k-mer matches against the reference rRNA database to generate candidate hits efficiently; (2) full Needleman-Wunsch pairwise alignment of the high-scoring segment pairs produced by the k-mer search to refine candidate reference matches, recording alignment score, pairwise identity, matches/mismatches/gaps and query/reference start-end positions; (3) a sensitive confidence algorithm that combines a score confidence (identity of the assigned taxonomy vs. the first sequence not matching it) with an identity-cutoff confidence using preoptimized per-level cutoffs (e.g. 0.95:0.05 at species level in the taxonomy header format). Run the result on a small rRNA amplicon-style query set against the reference, and write per-read mappings with confidence values bounded in [0,1] to a TSV. A combined confidence cutoff of 0.4–0.5 is recommended, as it yielded the highest F½-score in MAPseq's benchmarks.

## Related tools

- **MAPseq** (Reference implementation of the k-mer search with pre-clustering, Needleman-Wunsch HSP alignment and confidence estimation for rRNA classification) — https://github.com/jfmrod/mapseq
- **gcc** (Compiler used to build MAPseq from source via ./configure CC="gcc ..." and make)

## Examples

```
mapseq rawseqs.fa > rawseqs.fa.mseq
```

## Evaluation signals

- Output file per_read_confidence_scores.tsv exists with per-read rows containing taxonomy assignment and confidence fields
- All confidence values are bounded in [0,1]
- Implementation runs end-to-end as a self-contained reference (script_runs check)
- MSEQ output fields match the documented layout: query id, reference id, alignment score, pairwise identity, matches, mismatches, gaps, start/end positions, strand, then taxonomy/OTU levels with confidences
- Combined confidence cutoff of 0.4–0.5 reproduces the F½-score-optimal filtering used in MAPseq's benchmarks

## Limitations

- Accuracy and speed claims are relative to existing solutions for reference-based rRNA analysis; performance depends on reference database quality (v2.2b removed low-quality reference sequences that caused issues with low-quality queries)
- Second hits with missing taxonomic labels (uncertain annotation) can decrease the confidence of the top-hit assignment; the -ignoreEmptyTax option exists but was default-off pending benchmarks
- Confidence cutoff guidance (0.4 or 0.5) is benchmark-specific (highest F½-score) and may need re-optimization for other datasets

## Evidence

- [abstract] Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "Here we present MAPseq, a framework for reference-based rRNA sequence analysis that is up to 30% more accurate (F½ score) and up to one hundred times faster"
- [readme] The score confidence is calculated by comparing the identity of the assigned taxonomy to the identity of the first sequence not matching the assigned taxonomy.: [evidence span withheld — anchor status: external_doc]
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [readme] #cutoff: 0.00:0.08 0.70:0.35 0.70:0.35 0.70:0.35 0.80:0.25 0.92:0.08 0.95:0.05: [evidence span withheld — anchor status: external_doc]
