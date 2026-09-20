---
name: dereplicated-amplicon-sequence-processing
description: Use when a dereplicated, chimera-removed FASTA of amplicon reads (e.g.
  output of vsearch uchime-denovo) must be clustered into MOTUs with a fixed similarity
  threshold before building a MOTU table.
when_to_use_negative:
- Input reads have not yet been dereplicated and chimera-filtered (e.g. raw alignments
  not passed through obiuniq and vsearch uchime-denovo) — cluster only cleaned, dereplicated
  reads
- The goal is taxonomic assignment rather than MOTU delimitation — use ecotag downstream
  instead of re-clustering
- Species-level discrimination of closely related taxa is required (the article notes
  some MOTUs are 100% identical to multiple BOLD species, e.g. within Carcharhinus,
  Rhizoprionodon, Negaprion) — 99% clustering alone will not separate them
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: sumaclust
  role: clusters the dereplicated, chimera-filtered reads into MOTUs at a constant
    99% sequence similarity threshold
- name: OBITools
  role: framework used to extract MOTU representative sequences, aggregate read counts
    per MOTU, and write the MOTU table
- name: vsearch
  role: provides the uchime-denovo chimera removal step whose output is the input
    to sumaclust clustering
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:5e63f0d96c4e07cbf0b89382aaef6befbb91c40f8a08dd5c6d165ff5f9965e42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - sumaclust
  - vsearch
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# dereplicated-amplicon-sequence-processing

## Summary

Delimits molecular operational taxonomic units (MOTUs) from chimera-filtered, dereplicated amplicon reads by clustering sequences with sumaclust at a constant 99% similarity threshold, producing MOTU representative sequences for downstream taxonomic assignment.

## When to use

Use when a dereplicated, chimera-removed FASTA of amplicon reads (e.g. output of vsearch uchime-denovo) must be clustered into MOTUs with a fixed similarity threshold before building a MOTU table.

## When NOT to use

- Input reads have not yet been dereplicated and chimera-filtered (e.g. raw alignments not passed through obiuniq and vsearch uchime-denovo) — cluster only cleaned, dereplicated reads
- The goal is taxonomic assignment rather than MOTU delimitation — use ecotag downstream instead of re-clustering
- Species-level discrimination of closely related taxa is required (the article notes some MOTUs are 100% identical to multiple BOLD species, e.g. within Carcharhinus, Rhizoprionodon, Negaprion) — 99% clustering alone will not separate them

## Inputs

- chimera-filtered, dereplicated amplicon sequence dataset (FASTA, output of vsearch uchime-denovo; e.g. sequences from Mendeley dataset 489gckxbz3/1)
- sequence read counts associated with each dereplicated read

## Outputs

- MOTU representative-sequence table (e.g. motus_table.csv) containing one representative sequence, total read count, and MOTU identifier per delimited MOTU
- MOTU representative sequences ready for ecotag taxonomic assignment

## How to apply

Start from the chimera-filtered, dereplicated dataset produced by the vsearch uchime-denovo step (in this study, sequences deposited at Mendeley dataset 489gckxbz3/1). Run sumaclust with a fixed, constant sequence similarity threshold of 99% to delimit MOTUs from the dereplicated reads. Extract the representative sequence for each delimited MOTU and aggregate read counts per MOTU within the OBITools framework. Write the MOTU table (representative sequence, total read count, MOTU identifier) to a file such as motus_table.csv, which then feeds the single-read-MOTU removal step and ecotag taxonomic assignment. A constant 99% threshold is deliberately used rather than sample-specific or variable-threshold clustering, matching the paper's reported parameterization.

## Related tools

- **sumaclust** (clusters the dereplicated, chimera-filtered reads into MOTUs at a constant 99% sequence similarity threshold)
- **OBITools** (framework used to extract MOTU representative sequences, aggregate read counts per MOTU, and write the MOTU table)
- **vsearch** (provides the uchime-denovo chimera removal step whose output is the input to sumaclust clustering)

## Examples

```
sumaclust -t 0.99 chimera_filtered.fasta > motus_representatives.fasta
```

## Evaluation signals

- Output table contains exactly one representative sequence per delimited MOTU
- MOTU delimitation is consistent with a constant 99% similarity threshold as reported in the paper
- Per-MOTU read counts aggregate correctly from the dereplicated reads within the OBITools framework
- The MOTU table schema includes representative sequence, total read count, and MOTU identifier
- The table can be passed directly into the single-read-MOTU removal step and ecotag taxonomic assignment without reformatting

## Limitations

- Some MOTUs were 100% identical to sequences of multiple species in the BOLD database, so MOTU delimitation at 99% does not guarantee species-level resolution for closely related Carcharhinus, Rhizoprionodon and Negaprion
- Single-read MOTUs cannot be fully attributed to contamination or sequencing error and require a conservative downstream removal step
- Taxonomic resolution depends on the 127 bp COI fragment; the article notes longer-fragment primers would be needed to resolve closely related species

## Evidence

- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [other] Run sumaclust on the dereplicated reads with a fixed similarity threshold of 99% to delimit MOTUs.: [evidence span withheld — anchor status: not_found]
- [discussion] Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at Mendeley dataset 489gckxbz3/1.: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1."
- [discussion] the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database: "the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database"
