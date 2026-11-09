---
name: genbank-best-match-assignment
description: Use when OTUs must be assigned taxonomic identity against a public reference
  database (GenBank) to evaluate redundancy of clustering methods or to validate curation
  results for known taxa.
when_to_use_negative:
- When no reference sequences for the target taxon exist in GenBank — the article
  notes incomplete reference databases make errors hard to separate from true OTUs
- When the goal is to identify single rare species — the article states LULU curation
  may not be suitable for identifying single rare species in community data sets
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0080
tools:
- name: LULU
  role: Produces the curated_result object (original_table, curated_table, otu_map)
    whose OTUs are assigned to species via best GenBank match for validation
  repo: https://github.com/tobiasgf/lulu
- name: BLASTn
  role: Used for assigning OTUs to species in GenBank and producing the internal match
    list prior to LULU curation
- name: VSEARCH
  role: One of the clustering methods whose OTU tables are taxonomically evaluated
    via best GenBank match
- name: SWARM
  role: One of the clustering methods whose OTU tables are taxonomically evaluated
    via best GenBank match
  repo: https://github.com/frederic-mahe/swarm
- name: DADA2
  role: One of the clustering methods whose OTU tables are taxonomically evaluated
    via best GenBank match
- name: CROP
  role: One of the clustering methods whose OTU tables are taxonomically evaluated
    via best GenBank match
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-017-01312-x
    title: Algorithm for post-clustering curation of DNA amplicon data yields reliable
      biodiversity estimates
  source_content_hash: sha256:d3c07e812e2f180ce66b1e80411ef30678232f261354749b622a161eeaf80dd4
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
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# genbank-best-match-assignment

## Summary

Assigns OTUs to species by finding the best GenBank match of each OTU's representative (centroid) sequence, enabling taxonomic validation of clustering/curation outcomes — e.g., quantifying taxonomically redundant OTUs in single-species genera such as Fagus and Calluna.

## When to use

Use when OTUs must be assigned taxonomic identity against a public reference database (GenBank) to evaluate redundancy of clustering methods or to validate curation results for known taxa.

## When NOT to use

- When no reference sequences for the target taxon exist in GenBank — the article notes incomplete reference databases make errors hard to separate from true OTUs
- When the goal is to identify single rare species — the article states LULU curation may not be suitable for identifying single rare species in community data sets

## Inputs

- FASTA file of OTU representative/centroid sequences with IDs matching the OTU table (e.g. centroids_test.txt)
- OTU table (samples as columns, OTUs as rows) — pre-curation and/or LULU-curated
- LULU curated_result object (original_table, curated_table, otu_map with parent_id and curated status)
- GenBank reference database with ITS2 sequences for the target taxonomic group

## Outputs

- Per-OTU best GenBank match assignments (taxon identity and similarity to best match)
- Per-genus tabulation (e.g. CSV rows with OTU id, read counts before and after curation, best GenBank match, parent assignment)
- Counts of erroneous or taxonomically redundant OTUs per genus per clustering method

## How to apply

Produce or obtain a FASTA file with the representative (centroid) sequence of each OTU whose IDs match the OTU table row names. Search the OTU sequences against GenBank (ITS2 references covering the target group — the article's GenBank references contained ITS2 data for 88% of the Danish field-survey species) to identify the best-matching sequence for each OTU representative, and record the best GenBank match (taxon and similarity) per OTU. Use these assignments, e.g. all OTUs matching Fagus or Calluna, to tabulate per-taxon OTU counts and read counts before and after LULU curation (from curated_result$original_table, curated_result$curated_table, and curated_result$otu_map). Judge correctness by confirming that post-curation total read counts per genus are preserved while the number of OTUs collapses onto the dominant parent, yielding realistic diversity for genera containing one abundant species.

## Related tools

- **LULU** (Produces the curated_result object (original_table, curated_table, otu_map) whose OTUs are assigned to species via best GenBank match for validation) — https://github.com/tobiasgf/lulu
- **BLASTn** (Used for assigning OTUs to species in GenBank and producing the internal match list prior to LULU curation)
- **VSEARCH** (One of the clustering methods whose OTU tables are taxonomically evaluated via best GenBank match)
- **SWARM** (One of the clustering methods whose OTU tables are taxonomically evaluated via best GenBank match) — https://github.com/frederic-mahe/swarm
- **DADA2** (One of the clustering methods whose OTU tables are taxonomically evaluated via best GenBank match)
- **CROP** (One of the clustering methods whose OTU tables are taxonomically evaluated via best GenBank match)

## Examples

```
blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta
```

## Evaluation signals

- Post-curation total read counts per genus are preserved while OTU number collapses onto the dominant parent OTU
- Genera with only one species represented (Fagus, Calluna) show high OTU counts pre-curation for most methods and realistic diversity levels after LULU curation
- Every OTU in the tabulation carries a best GenBank match consistent with its representative sequence
- Parent-daughter assignments in the per-genus CSV rows are consistent with curated_result$otu_map bottom-to-top merging
- Similarity between OTU and best GenBank match and taxonomic redundancy can be compared across VSEARCH, SWARM, CROP, DADA2 tables before and after curation

## Limitations

- GenBank reference coverage may be incomplete — ITS2 data existed for only 88% of the field-survey species, so some OTUs cannot be confidently assigned
- PCR/sequencing errors and intraspecific variation inflate sequence richness beyond true richness, complicating taxonomic interpretation of pre-curation OTU counts
- CROP OTU picking appeared suboptimal (low similarity to best GenBank match, poor redundancy measures), affecting taxonomic assignments for that method

## Evidence

- [intro] For additional parts of the validation procedure, we assigned OTUs to species in public reference databases (GenBank): "For additional parts of the validation procedure, we assigned OTUs to species in public reference databases (GenBank)"
- [intro] GenBank ITS2 coverage of field species: "we assigned OTUs to species in public reference databases (GenBank), which contained ITS2 data for 88% of species recorded in the field survey"
- [results] Fagus/Calluna redundancy and curation finding: [evidence span withheld — anchor status: not_found]
- [discussion] Bottom-to-top merging preserving read counts: "After parsing the full OTU table, a new OTU table is constructed by merging read counts of errors with their designated parent OTU from bottom to top"
- [discussion] Match list production via BLASTn: [evidence span withheld — anchor status: not_found]
- [readme] BLASTn match-list command from README: "blastn -db OTU_sequences.fasta -outfmt '6 qseqid sseqid pident' -out match_list.txt -qcov_hsp_perc 80 -perc_identity 84 -query OTU_sequences.fasta"
