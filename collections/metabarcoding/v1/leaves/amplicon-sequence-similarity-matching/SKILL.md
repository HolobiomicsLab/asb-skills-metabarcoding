---
name: amplicon-sequence-similarity-matching
description: Use when an OTU table plus its representative centroid sequences (e.g.
  otutable_test.txt and centroids_test.txt) must be converted into a match list of
  internal OTU similarities before running LULU curation.
when_to_use_negative:
- You have no OTU centroid/representative sequences — LULU requires a match list produced
  from pairwise comparison of the OTU sequences, not just the count table
- You need de novo OTU inference or clustering itself — this skill only computes internal
  pairwise similarities of already-defined OTUs (clustering is done by VSEARCH, SWARM,
  CROP or DADA2 beforehand)
- Your target group's average intraspecific variation makes an 84% identity floor
  inappropriate without adjustment — the threshold must address the intraspecific
  variation of the target group
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: BLASTn
  role: 'Primary tool for producing the match list: makeblastdb builds a nucleotide
    database of OTU centroid sequences and blastn self-aligns them with -outfmt ''6
    qseqid sseqid pident'', -qcov_hsp_perc 80, -perc_identity 84'
- name: VSEARCH
  role: Alternative to BLASTn for internal OTU matching via --usearch_global against
    a self database with --id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits
    10
- name: LULU
  role: 'Downstream consumer of the match list: the lulu() R function requires the
    match list as its second input to identify and merge erroneous ''daughter'' OTUs
    into ''parent'' OTUs'
  repo: https://github.com/tobiasgf/lulu
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:419b311889639449678a83781b2a2da3bc3313c697db4fed5ef4dc63c732c729
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - BLASTn
  - LULU
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# amplicon-sequence-similarity-matching

## Summary

Produces a three-column match list of internal pairwise similarities between all OTU centroid sequences (query id, target id, percent identity), which is the required co-occurrence-curation input for LULU post-clustering curation of amplicon OTU tables.

## When to use

Use when an OTU table plus its representative centroid sequences (e.g. otutable_test.txt and centroids_test.txt) must be converted into a match list of internal OTU similarities before running LULU curation.

## When NOT to use

- You have no OTU centroid/representative sequences — LULU requires a match list produced from pairwise comparison of the OTU sequences, not just the count table
- You need de novo OTU inference or clustering itself — this skill only computes internal pairwise similarities of already-defined OTUs (clustering is done by VSEARCH, SWARM, CROP or DADA2 beforehand)
- Your target group's average intraspecific variation makes an 84% identity floor inappropriate without adjustment — the threshold must address the intraspecific variation of the target group

## Inputs

- FASTA file of representative centroid sequences for each OTU (e.g. centroids_test.txt), with OTU ids matching the OTU table
- OTU table with samples as columns and unique OTU ids as row names (e.g. otutable_test.txt, 2425 OTUs over 130 samples)
- Similarity thresholds: minimum percent identity (84%), query coverage (80% HSP for BLASTn, 0.9 for VSEARCH), maxhits (10 for VSEARCH)

## Outputs

- match_list.txt: a three-column, whitespace/tab-delimited list of query OTU id, matching OTU id, and percent identity (e.g. 'OTU1 OTU2 90.000')

## How to apply

Build a BLAST database from the OTU centroid FASTA with makeblastdb (-parse_seqids -dbtype nucl), then run blastn with -outfmt '6 qseqid sseqid pident', -qcov_hsp_perc 80 and -perc_identity 84 to emit a match_list.txt of query id, target id, and percent identity (equivalently VSEARCH --usearch_global --db self --id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits 10 --userfields query+target+id). The match list must have exactly three columns: query OTU id, matching OTU id, and percent similarity, with ids matching the OTU table row names. The 84% identity floor acts as the dissimilarity threshold that LULU later applies when designating 'hits' for each 'potential daughter'; a lower threshold in LULU will have no effect if the match list was produced with a minimum match %. The resulting match list, together with the OTU table (samples as columns, OTUs as rows with unique row names), is supplied to the lulu() R function as its second required input.

## Related tools

- **BLASTn** (Primary tool for producing the match list: makeblastdb builds a nucleotide database of OTU centroid sequences and blastn self-aligns them with -outfmt '6 qseqid sseqid pident', -qcov_hsp_perc 80, -perc_identity 84)
- **VSEARCH** (Alternative to BLASTn for internal OTU matching via --usearch_global against a self database with --id .84 --iddef 1 --query_cov .9 --maxaccepts 0 --maxhits 10)
- **LULU** (Downstream consumer of the match list: the lulu() R function requires the match list as its second input to identify and merge erroneous 'daughter' OTUs into 'parent' OTUs) — https://github.com/tobiasgf/lulu

## Examples

```
makeblastdb -in OTU_sequences.fasta -parse_seqids -dbtype nucl; blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta
```

## Evaluation signals

- Match list has exactly three columns: query OTU id, target OTU id, and percent similarity (with or without decimals, e.g. 'OTU3 OTU3 100.000')
- All ids in the match list match the OTU id row names of the OTU table
- No reported match falls below the minimum identity threshold used (84% for BLASTn -perc_identity, or 0.84 with VSEARCH --iddef 1)
- The OTU's own self-match (100% identity) appears in the list, as in the README example ('OTU3 OTU3 100.000')
- Downstream validation: running LULU with defaults (minimum_match 84, minimum_relative_cooccurence 0.95, minimum_ratio 1) on the tutorial data yields curated_count = 459 and discarded_count = 1966

## Limitations

- If the match list was produced with a minimum match %, setting a lower dissimilarity threshold in LULU afterwards will have no effect
- Match list generation is quadratic in dataset size for pairwise matching approaches, so very large amplicon sets are computationally costly
- LULU curation downstream may not be suitable for identifying single rare species in community data sets, and the cost of curation is a small loss of real (rare, low-abundance) species

## Evidence

- [discussion] The workflow (Fig. 2) requires (1) an OTU table ... and (2) a so-called match list—a list with the most similar OTUs ... produced prior to the LULU curation by an external algrothm (e.g., BLASTn or: [evidence span withheld — anchor status: not_found]
- [readme] The only requirements is that the match list has three columns with pair wise similarity scores for the OTUs.: "The only requirements is that the match list has three columns with pair wise similarity scores for the OTUs. The first column contains the id of the query OTU, the second column contains the id of"
- [methods] Select all OTUs in the match list matching the potential daughter within the dissimilarity threshold: "Select all OTUs in the match list matching the potential daughter within the dissimilarity threshold selected, and designate these as 'hits'."
- [readme] 3. Select all OTUs in the match list matching the potential daughter within the dissimilarity threshold selected: "The dissimilarity threshold can be set to anything, but if the matchlist was produced with a minimum match %, a lower threshold will not have an effect."
