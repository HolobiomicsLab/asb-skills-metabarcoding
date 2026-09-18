---
name: cross-database-taxonomic-overlap-visualization
description: Use when two or more COI reference databases (GenBank- vs BOLD-derived)
  must be compared for overlapping and unique taxonomic names at Class–Species ranks,
  to establish whether they are complementary rather than redundant.
when_to_use_negative:
- The goal is to benchmark taxonomic classifier accuracy (e.g. vsearch-SINTAX vs q2-feature-classifier
  vs the BOLD Classification API) rather than compare database composition — that
  requires classified mock-community and bat-guano queries, not per-rank name tabulation.
- The sequences have already been dereplicated and collapsed with taxonomy intact,
  so re-running Porter dereplication (pick_otus.py at 1.0 similarity, vsearch --derep_fulllength)
  would be redundant.
- Only a single database is available — shared/unique name counts and upset-set visualization
  require at least two databases to compare.
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0196
tools:
- name: seqkit
  role: Filters Palmer fasta to arthropod records via regex 'p:Arthropoda' on headers
- name: VSEARCH
  role: Converts Palmer COI.udb to fasta (--udb2fasta) and dereplicates Porter arthropod
    sequences (--derep_fulllength) to 515,780 distinct sequences
  repo: https://github.com/torognes/vsearch
- name: QIIME1 pick_otus.py
  role: Clusters identical Porter sequences at --similarity 1.0 during dereplication
    before LCA consensus taxonomy assignment
- name: create_consensus_taxonomy.py
  role: Applies LCA consensus taxonomy to clustered identical Porter sequences during
    dereplication
  repo: https://gist.github.com/walterst/bd69a19e75748f79efeb
- name: R (figure4_database_comparisons.R)
  role: Tabulates shared/distinct taxonomic names per rank (Class–Species) and missing-data
    counts, then renders the upset plot
  repo: https://github.com/devonorourke/tidybug
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1002/ece3.6594
    title: A total crapshoot? Evaluating bioinformatic decisions in animal diet metabarcoding
      analyses
  source_content_hash: sha256:7f673a49240845a4e9b5b0409f90c61825108905ed3b80a5f6e76912f11fc776
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/torognes/vsearch
  tools:
  - QIIME1 pick_otus.py
  - R
  - R (figure4_database_comparisons.R)
  - VSEARCH
  - create_consensus_taxonomy.py
  - csvtk
  - seqkit
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.18181818181818182
n_claims: 11
---

# Cross-database taxonomic overlap visualization

## Summary

Quantifies shared and unique taxonomic names (Class through Species) across the Porter (GenBank-derived), Palmer (amptk COI), and tidybug (BOLD-derived) arthropod COI reference databases, and renders the overlap as the Figure 4B upset plot to demonstrate that GenBank- and BOLD-derived resources are complementary rather than redundant.

## When to use

Use when two or more COI reference databases (GenBank- vs BOLD-derived) must be compared for overlapping and unique taxonomic names at Class–Species ranks, to establish whether they are complementary rather than redundant.

## When NOT to use

- The goal is to benchmark taxonomic classifier accuracy (e.g. vsearch-SINTAX vs q2-feature-classifier vs the BOLD Classification API) rather than compare database composition — that requires classified mock-community and bat-guano queries, not per-rank name tabulation.
- The sequences have already been dereplicated and collapsed with taxonomy intact, so re-running Porter dereplication (pick_otus.py at 1.0 similarity, vsearch --derep_fulllength) would be redundant.
- Only a single database is available — shared/unique name counts and upset-set visualization require at least two databases to compare.

## Inputs

- Porter Eukaryote CO1 v3.2 training fasta from the CO1Classifier GitHub release
- Palmer amptk COI database (COI.udb, converted to fasta via vsearch --udb2fasta)
- tidybug dereplicated BOLD files (boldCOI.derep.fasta, boldCustom.allArth) from OSF deposit https://osf.io/k3eh6/

## Outputs

- Per-rank taxa tables (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz) with Class-through-Species strings and per-rank missing-data counts
- Counts of shared and unique taxonomic names per rank (e.g. 12,788 shared genera; 4,251 Porter-only and 3,885 tidybug-only genera; 362 Porter-only and 248 tidybug-only families)
- figure4_upsetPlot.png upset plot of taxonomic-name overlap across the three databases

## How to apply

Restrict each database to arthropod records first — filter Palmer with `seqkit grep` on 'p:Arthropoda' (1,617,885 → 1,565,831 records) and Porter with `grep ';Arthropoda;'` (1,280,577 → 883,979 records); convert Palmer's COI.udb to fasta with `vsearch --udb2fasta`. Dereplicate Porter arthropod sequences by stripping taxonomy from headers, clustering identical sequences with pick_otus.py at --similarity 1.0, applying the LCA consensus script create_consensus_taxonomy.py, and collapsing with `vsearch --derep_fulllength` (to 515,780 distinct sequences). Extract Class-through-Species taxonomy strings from each database into taxa tables (palmer.taxa.txt.gz, porter.taxa.txt.gz, derep.taxa.txt.gz), discarding redundant kingdom/phylum fields. Tabulate shared and distinct taxonomic names per rank with figure4_database_comparisons.R, counting records with missing data at each rank, then render the overlap as the figure4_upsetPlot.png upset plot.

## Related tools

- **seqkit** (Filters Palmer fasta to arthropod records via regex 'p:Arthropoda' on headers)
- **VSEARCH** (Converts Palmer COI.udb to fasta (--udb2fasta) and dereplicates Porter arthropod sequences (--derep_fulllength) to 515,780 distinct sequences) — https://github.com/torognes/vsearch
- **QIIME1 pick_otus.py** (Clusters identical Porter sequences at --similarity 1.0 during dereplication before LCA consensus taxonomy assignment)
- **create_consensus_taxonomy.py** (Applies LCA consensus taxonomy to clustered identical Porter sequences during dereplication) — https://gist.github.com/walterst/bd69a19e75748f79efeb
- **R (figure4_database_comparisons.R)** (Tabulates shared/distinct taxonomic names per rank (Class–Species) and missing-data counts, then renders the upset plot) — https://github.com/devonorourke/tidybug

## Examples

```
vsearch --derep_fulllength tmp_nolabels.fasta
```

## Evaluation signals

- Arthropod filter record counts reproduce: Palmer 1,617,885 → 1,565,831 and Porter 1,280,577 → 883,979 records
- Porter dereplication collapses to exactly 515,780 distinct arthropod sequences
- Genera-level overlap reproduces: 12,788 shared genera, 4,251 unique to Porter, 3,885 unique to tidybug
- Unique family fractions reproduce: 362 Porter-only and 248 tidybug-only families
- The rendered upset plot shows each of the three databases contributing both shared and unique taxonomic names at every rank from Class through Species, consistent with the complementarity conclusion

## Limitations

- BOLD discontinued packaged versioned releases in 2015, so the tidybug BOLD-derived database is tied to a specific download date (BOLD records accessed 24 February 2019 via the bold R package) and cannot be exactly reproduced later without version ambiguity
- There is no single curated and versioned COI reference resource widely used in animal diet studies, making cross-study comparisons of database composition difficult
- The tidybug comparison made no attempt to determine how pervasive missing Phylum/Class information is in BOLD records, so completeness metrics at those ranks may be under-counted

## Evidence

- [abstract] available reference datasets from GenBank and BOLD for the animal marker gene cytochrome oxidase I (COI) can be complementary: "available reference datasets from GenBank and BOLD for the animal marker gene cytochrome oxidase I (COI) can be complementary"
- [intro] examined reference database choice and filtering criteria effects: "Thus, we examined how the effects of reference database choice and subsequent filtering criteria (clustering radius) on reference composition"
- [methods] scripts and data files available in the tidybug repo: "Complete details including scripts and data files used in this manuscript are available at the GitHub repo for this project: https://github.com/devonorourke/tidybug"
- [discussion] no curated versioned resource complicates comparisons: "yet there is no single curated and versioned resource widely used in animal diet studies, making comparisons among studies extremely difficult"
