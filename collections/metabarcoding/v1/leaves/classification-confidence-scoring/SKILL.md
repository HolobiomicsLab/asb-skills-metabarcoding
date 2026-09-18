---
name: classification-confidence-scoring
description: Use when assigning rRNA reads to taxonomy/OTU levels and you need calibrated
  per-read confidence values (bounded in [0,1]) to filter classifications, e.g. applying
  a combined confidence cutoff of 0.4-0.5.
when_to_use_negative:
- Input is not rRNA sequence data — MAPseq is designed for reference-based ribosomal
  RNA analysis with full-length rRNA references and pre-generated OTU clusters
- You need de novo OTU clustering without a reference database — this skill requires
  a taxonomy-annotated reference set
- You only need a count table, not per-read confidences — use mapseq -otucounts/-otutable
  summaries instead of re-deriving confidences
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0659
tools:
- name: MAPseq
  role: Reference implementation whose confidence algorithm converts alignment evidence
    into per-read combined confidence scores for NCBI taxonomy and OTU mappings
  repo: https://github.com/jfmrod/mapseq
- name: gcc
  role: Compiler used to build MAPseq from source (./configure CC="gcc ...") when
    reimplementing or validating the confidence stage
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioinformatics/btx517
    title: 'MAPseq: highly efficient k-mer search with confidence estimates, for rRNA
      sequence analysis'
  source_content_hash: sha256:ca4f96d9e63461a294fd4a7e278b9e3794f3cfdc2f45ec0973c52dd5e2fbd1bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/jfmrod/mapseq
  tools:
  - MAPseq
  - gcc
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# classification-confidence-scoring

## Summary

MAPseq's per-read confidence computation converts Needleman-Wunsch alignment evidence into combined confidence scores for taxonomy and hierarchical OTU assignments, controlling misclassification by combining a score confidence with an identity-cutoff confidence at each taxonomic level.

## When to use

Use when assigning rRNA reads to taxonomy/OTU levels and you need calibrated per-read confidence values (bounded in [0,1]) to filter classifications, e.g. applying a combined confidence cutoff of 0.4-0.5.

## When NOT to use

- Input is not rRNA sequence data — MAPseq is designed for reference-based ribosomal RNA analysis with full-length rRNA references and pre-generated OTU clusters
- You need de novo OTU clustering without a reference database — this skill requires a taxonomy-annotated reference set
- You only need a count table, not per-read confidences — use mapseq -otucounts/-otutable summaries instead of re-deriving confidences

## Inputs

- FASTA file of demultiplexed, quality-filtered rRNA reads (amplicon or shotgun)
- Reference nucleotide FASTA of full-length rRNA sequences (e.g. the default mapref reference)
- Tab-separated taxonomy file(s) with '#cutoff', '#name', '#levels' header and accession-to-taxonomy rows
- HSP Needleman-Wunsch alignment results (alignment score, pairwise identity, matches, mismatches, gaps)

## Outputs

- .mseq output in 'confidences' format with per-read taxonomy assignments and confidence values at each taxonomic and OTU level
- per_read_confidence_scores.tsv with per-read rows containing taxonomy assignment and bounded confidence fields

## How to apply

After the k-mer search with pre-clustering generates candidate references and full Needleman-Wunsch alignment produces alignment scores, identity, matches, mismatches and gaps for each read, compute the classification confidence for each taxonomic level. The combined confidence is the product of a score confidence and an identity-cutoff confidence: the score confidence compares the pairwise identity of the assigned taxonomy to the identity of the first reference sequence not matching the assigned taxonomy (controlling misclassification errors), while the identity-cutoff confidence applies preoptimized cutoffs per taxonomic level (e.g. the '#cutoff' values like 0.00:0.08, 0.92:0.08, 0.95:0.05 in the taxonomy file header) to estimate the probability that the query is not too divergent from the assigned taxonomy. Output per-read rows in the 'confidences' format of the .mseq file, with each taxonomy classification followed by its confidence, separated by empty fields. Apply a combined confidence cutoff of 0.4 or 0.5 when calling assignments, since this yielded the highest F1/2-score in MAPseq's benchmarks. Write the per-read mapping table with confidence fields to per_read_confidence_scores.tsv for validation.

## Related tools

- **MAPseq** (Reference implementation whose confidence algorithm converts alignment evidence into per-read combined confidence scores for NCBI taxonomy and OTU mappings) — https://github.com/jfmrod/mapseq
- **gcc** (Compiler used to build MAPseq from source (./configure CC="gcc ...") when reimplementing or validating the confidence stage)

## Examples

```
mapseq rawseqs.fa --outfmt confidences > rawseqs.fa.mseq
```

## Evaluation signals

- per_read_confidence_scores.tsv exists with one row per read containing taxonomy assignment and confidence fields
- All confidence values are bounded in [0,1]
- Simple-format assignments only appear where combined confidence is at least 0.5 (or chosen 0.4 cutoff)
- The pipeline runs end-to-end as a self-contained reference (script_runs check)
- Confidence decreases at deeper taxonomic levels for divergent queries, mirroring the identity-cutoff mechanism (e.g. Ruminococcus gnavus at 0.06 vs Clostridiales at 0.55 in the documented example)

## Limitations

- Second hits with missing taxonomic labels (uncertain annotation) can decrease the confidence of the top-hit assignment; the -ignoreEmptyTax option (added in v1.2.4, off by default) prevents this but awaits thorough benchmarking
- A recommended combined confidence cutoff of 0.4-0.5 is benchmark-dependent (highest F1/2-score in MAPseq's benchmarks); other cutoffs may misclassify
- The identity-cutoff confidence depends on preoptimized per-level cutoffs in the reference taxonomy file header; custom references must supply appropriate cutoff values
- LTP taxonomy was dropped from the default reference due to low coverage

## Evidence

- [readme] The combined confidence is computed based on a score confidence, used to control misclassification errors, and a identity cutoff confidence, used to ensure that the query isnt misclassified due to the inexistence of a sequence representative in the database: [evidence span withheld — anchor status: external_doc]
- [readme] We recommend using a combined confidence cutoff of 0.4, or 0.5 as this value yielded the highest F1/2-score for MAPseq in our benchmarks.: [evidence span withheld — anchor status: external_doc]
- [abstract] up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions: "up to 30% more accurate (F½ score) and up to one hundred times faster than existing solutions"
- [readme] The identity cutoff confidence uses preoptimized cutoffs at each taxonomic level to calculate the confidence that the query is not too divergent from the assigned taxonomy.: [evidence span withheld — anchor status: external_doc]
