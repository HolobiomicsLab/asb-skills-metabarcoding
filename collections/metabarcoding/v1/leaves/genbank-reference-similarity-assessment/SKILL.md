---
name: genbank-reference-similarity-assessment
description: Use when validating OTU tables by comparing OTU representative sequences
  against public reference data (GenBank) to quantify average -match similarity, e.g.
  as a curation-effect metric across clustering levels.
when_to_use_negative:
- When no adequate reference sequences exist for the target taxon — the study notes
  GenBank held ITS2 data for only 88% of surveyed species, so unrepresented taxa cannot
  be assessed this way
- When the aim is reference-independent curation — LULU itself 'does not require a
  sequence reference database'; this skill is only the reference-based validation
  metric
- When evaluating internal OTU-OTU redundancy or co-occurrence structure rather than
  similarity to external reference sequences
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_3697
tools:
- name: LULU
  role: Produces the curated OTU tables whose average best GenBank match similarity
    is compared against raw tables as a curation-effect metric
  repo: https://github.com/tobiasgf/lulu
- name: VSEARCH
  role: Can produce the internal match list (query+target+id with --id .84 --iddef
    1) used in the matching workflow; also one of the clustering methods whose tables
    are assessed
- name: SWARM
  role: Clustering method whose OTU tables are evaluated for average best GenBank
    match similarity before and after LULU curation
  repo: https://github.com/torognes/swarm
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:abb226332dd2e974d3a6876ed0ae1506d31fa0ac1b7f655359a2b8c2cf716eed
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/lulu
  tools:
  - CROP
  - DADA2
  - LULU
  - R
  - SWARM
  - VSEARCH
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# genbank-reference-similarity-assessment

## Summary

Assess OTU inference quality by computing the average similarity between each OTU's representative sequence and its best match in the GenBank reference database, used as a Table 1 validation metric before and after LULU curation across clustering methods (VSEARCH, SWARM, CROP, DADA2).

## When to use

Use when validating OTU tables by comparing OTU representative sequences against public reference data (GenBank) to quantify average best-match similarity, e.g. as a curation-effect metric across clustering levels.

## When NOT to use

- When no adequate reference sequences exist for the target taxon — the study notes GenBank held ITS2 data for only 88% of surveyed species, so unrepresented taxa cannot be assessed this way
- When the aim is reference-independent curation — LULU itself 'does not require a sequence reference database'; this skill is only the reference-based validation metric
- When evaluating internal OTU-OTU redundancy or co-occurrence structure rather than similarity to external reference sequences

## Inputs

- FASTA file of OTU representative (centroid) sequences with ids matching the OTU table
- OTU table (samples as columns, unique OTU ids as row names), raw and LULU-curated versions per clustering method/level
- GenBank reference database containing ITS2 sequences (88% coverage of surveyed plant species in the study)

## Outputs

- Three-column match list (query OTU id, matching reference id, percent similarity) per OTU table
- Per-table 'average best GenBank match' similarity values (one per method/level/curation-status), assembled into table1_validation_metrics.csv

## How to apply

Build a three-column match list (query OTU id, matching OTU id, percent similarity) by searching OTU representative sequences against a reference database; the LULU paper used GenBank, which contained ITS2 data for 88% of the 564 plant species recorded in the Danish field survey. Produce the match list with BLASTn (e.g. makeblastdb plus blastn with -outfmt '6 qseqid sseqid pident', -qcov_hsp_perc 80, -perc_identity 84) or VSEARCH (--usearch_global with --id .84 --iddef 1 --userfields query+target+id), ensuring OTU ids match those of the OTU table (samples as columns, OTUs as rows). For each OTU table, take the best (highest percent identity) reference hit per OTU and average these similarities to obtain the 'average best GenBank match' metric. Compute this metric separately for raw and LULU-curated versions of each method/level combination (VSEARCH 98.5-95%, SWARM 99-95%, CROP 98-95%, DADA2 100%, DADA2+VS 98.5-95%) and assemble it into a Table 1-equivalent CSV. Curation should raise average best-match similarity because erroneous OTUs merged with parents were disproportionately poor reference matches.

## Related tools

- **LULU** (Produces the curated OTU tables whose average best GenBank match similarity is compared against raw tables as a curation-effect metric) — https://github.com/tobiasgf/lulu
- **VSEARCH** (Can produce the internal match list (query+target+id with --id .84 --iddef 1) used in the matching workflow; also one of the clustering methods whose tables are assessed)
- **SWARM** (Clustering method whose OTU tables are evaluated for average best GenBank match similarity before and after LULU curation) — https://github.com/torognes/swarm

## Examples

```
makeblastdb -in OTU_sequences.fasta -parse_seqids -dbtype nucl; blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta
```

## Evaluation signals

- Average best GenBank match similarity should increase after LULU curation relative to the raw table for the same method/level combination
- Match list must have exactly three columns (query OTU id, matching OTU id, similarity score in percent, with or without decimals)
- OTU ids in the FASTA/match list must match the OTU table row names exactly
- Cross-method comparison should reproduce the paper's pattern that CROP shows notably low similarity to best GenBank match, indicating suboptimal OTU picking
- Similarity values are reported as percentages (e.g. '90.000') consistent with the match-list format

## Limitations

- Incomplete reference databases mean some OTUs lack a true best match; the study's GenBank held ITS2 data for only 88% of field-survey species
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match), so low similarity may reflect the clustering method rather than curation failure
- PCR/sequencing errors and intraspecific variation inflate sequence richness and blur the boundary between erroneous and true OTUs when interpreting similarity values

## Evidence

- [intro] GenBank reference database coverage: "we assigned OTUs to species in public reference databases (GenBank), which contained ITS2 data for 88% of species recorded in the field survey"
- [discussion] GenBank similarity as validation metric: "the similarity between OTU and best GenBank match (Fig. 1e) were far lower, and the taxonomic redundancy (Fig. 1c)"
- [discussion] CROP suboptimal: "the OTU picking of CROP seemed to be suboptimal compared to the other methods investigated"
- [readme] Match list format: "The only requirements is that the match list has three columns with pair wise similarity scores for the OTUs."
- [readme] BLASTn match-list command: "blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta"
