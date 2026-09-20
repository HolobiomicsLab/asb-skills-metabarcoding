---
name: matchlist-generation-from-otu-sequences
description: Use when you have an OTU table plus a FASTA file of OTU representative
  sequences and need to curate the table with LULU, which cannot run without a precomputed
  match list.
when_to_use_negative:
- You only need curated biodiversity metrics and already possess a valid match list
  — regenerating it is unnecessary
- The OTU ids in the sequence FASTA do not match the OTU table row names — fix the
  ids first, since LULU requires matching identifiers
- You are running dbotu3 in its intended one-step clustering mode, which does not
  use a LULU-style match list input
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_0610
tools:
- name: BLASTn
  role: Reference tool used in the LULU validation to produce the match list via makeblastdb
    + blastn with qcov_hsp_perc 80 and perc_identity 84
- name: VSEARCH
  role: Alternative pairwise matcher (--usearch_global --self) generating the match
    list consumed by the lulu() R function; also one of the initial clustering methods
    (with SWARM, CROP, DADA2) whose tables are curated
- name: LULU
  role: 'Downstream consumer of the match list: lulu(otutable, matchlist) merges co-occurring,
    sequence-similar daughter OTUs into parents'
  repo: https://github.com/tobiasgf/lulu
- name: SWARM
  role: One of the initial OTU clustering methods (VSEARCH, SWARM, CROP, DADA2) whose
    OTU tables and centroid sequences feed the matchlist-plus-LULU workflow
  repo: https://github.com/frederic-mahe/swarm
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:cebc900ce8dfe4a081967dc3ec2c0376c2a5d642b068ae91edcb9c030cdf0dca
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - BLASTn
  - CROP
  - DADA2
  - LULU
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_0610
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Matchlist generation from OTU sequences

## Summary

Produces a three-column match list (query OTU id, matching OTU id, percent similarity) by all-against-all pairwise comparison of OTU representative sequences, as the mandatory second input required by the LULU post-clustering curation algorithm.

## When to use

Use when you have an OTU table plus a FASTA file of OTU representative sequences and need to curate the table with LULU, which cannot run without a precomputed match list.

## When NOT to use

- You only need curated biodiversity metrics and already possess a valid match list — regenerating it is unnecessary
- The OTU ids in the sequence FASTA do not match the OTU table row names — fix the ids first, since LULU requires matching identifiers
- You are running dbotu3 in its intended one-step clustering mode, which does not use a LULU-style match list input

## Inputs

- FASTA file with one representative (centroid) sequence per OTU, ids matching the OTU table row names
- OTU table with samples as columns, OTUs as rows and unique OTU ids as row names (e.g. centroids_test.txt / otutable_test.txt with 2425 OTUs over 130 samples)

## Outputs

- match_list.txt: three-column table of query OTU id, matching OTU id, and percent similarity score for all internal OTU matches

## How to apply

Take the OTU representative sequences (centroids) whose ids exactly match the row names of the OTU table and compare the file against itself. With BLASTn, first build a database with makeblastdb, then run blastn with '-outfmt 6 qseqid sseqid pident', '-qcov_hsp_perc 80' and '-perc_identity 84' to write match_list.txt; alternatively use VSEARCH's --usearch_global with '--self --id .84 --iddef 1 --query_cov .9 --maxhits 10' to emit query+target+id. The output must have exactly three columns: query OTU id, matching OTU id, and similarity score in percent (with or without decimals). This match list, together with the OTU table (samples as columns, OTUs as rows), is then supplied to the lulu() R function, which uses the pairwise similarities to select 'hits' within the dissimilarity threshold when evaluating co-occurrence-based parent/daughter merging.

## Related tools

- **BLASTn** (Reference tool used in the LULU validation to produce the match list via makeblastdb + blastn with qcov_hsp_perc 80 and perc_identity 84)
- **VSEARCH** (Alternative pairwise matcher (--usearch_global --self) generating the match list consumed by the lulu() R function; also one of the initial clustering methods (with SWARM, CROP, DADA2) whose tables are curated)
- **LULU** (Downstream consumer of the match list: lulu(otutable, matchlist) merges co-occurring, sequence-similar daughter OTUs into parents) — https://github.com/tobiasgf/lulu
- **SWARM** (One of the initial OTU clustering methods (VSEARCH, SWARM, CROP, DADA2) whose OTU tables and centroid sequences feed the matchlist-plus-LULU workflow) — https://github.com/frederic-mahe/swarm

## Examples

```
vsearch --usearch_global OTU_sequences.fasta --db OTU_sequences.fasta --self --id .84 --iddef 1 --userout match_list.txt -userfields query+target+id --maxaccepts 0 --query_cov .9 --maxhits 10
```

## Evaluation signals

- Output file has exactly three whitespace-separated columns: query id, target id, percent identity (e.g. 'OTU1  OTU2  90.000')
- Every OTU id in the match list corresponds to an id in the OTU table / FASTA, and each OTU matches itself at ~100%
- Match percent values are on a 0–100 scale (percent, not fraction) because VSEARCH was run with --iddef 1
- Minimum identity floor is respected: BLASTn run with -perc_identity 84 or VSEARCH with --id .84, so no match below ~84% appears
- Downstream sanity check: lulu(otutable, matchlist) returns a curated table whose OTU count is reduced while total read counts per sample are preserved through parent merging

## Limitations

- The dissimilarity threshold used in LULU cannot go lower than the minimum match percent embedded in the match list — a stricter (lower) LULU threshold has no effect if the match list was pre-filtered at 84% identity
- Curation may merge or remove real rare species ('cost of curation'), and LULU curation may not be suitable for identifying single rare species
- Match list quality depends on representative sequence quality; erroneous OTUs from PCR/sequencing errors inflate sequence richness before curation

## Evidence

- [readme] readme: "The match list can in practice be produced with any tool for pair wise matching of sequences."
- [discussion] discussion: [evidence span withheld — anchor status: not_found]
- [intro] intro: [evidence span withheld — anchor status: not_found]
