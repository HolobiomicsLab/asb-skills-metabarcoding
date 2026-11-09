---
name: taxonomy-table-integration-with-read-counts
description: Use when you have 99%-similarity MOTU representative sequences (post
  sumaclust, single-read MOTUs removed) and need each MOTU assigned to the most specific
  consistent taxon and merged with per-sample read counts into one table.
when_to_use_negative:
- Input is raw or unclustered reads — taxonomic assignment should be done on MOTU
  representative sequences, not individual reads
- Reference database is not formatted for OBITools (no obitax taxonomy); ecotag cannot
  resolve taxon names without it
- The marker region resolves species uniquely (no 100% multi-species matches) — then
  simple best-hit assignment suffices and MOTU splitting is unnecessary
edam_operation: http://edamontology.org/operation_0360
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0080
tools:
- name: ecotag
  role: Assigns each MOTU representative sequence the most specific consistent taxon
    against the formatted reference database
  repo: github.com/metabarpark/Reference-databases
- name: OBITools
  role: Suite used to format the reference database (obitax/ecoPCR-format) and to
    run ecotag
  repo: github.com/metabarpark/Reference-databases
- name: ecoPCR
  role: Used for the in silico PCR against EMBL-EBI release R117 that supplies the
    non-elasmobranch part of the combined reference database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:97868ab61f90720b67e2d6b10e1d1089d7ccd8e2084ac08fdc79d8a602f6b817
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecoPCR
  - ecotag
  edam_operation: http://edamontology.org/operation_0360
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# Taxonomy table integration with read counts

## Summary

Assigns MOTU representative sequences to taxa by running ecotag (OBITools) against a BOLD-derived COI reference database, then aggregates the ecotag output into a per-MOTU table joining MOTU IDs, per-sample read counts, and assigned taxonomy. This is the step that turns delimited MOTUs into a taxonomically interpretable abundance table for eDNA metabarcoding studies.

## When to use

Use when you have 99%-similarity MOTU representative sequences (post sumaclust, single-read MOTUs removed) and need each MOTU assigned to the most specific consistent taxon and merged with per-sample read counts into one table.

## When NOT to use

- Input is raw or unclustered reads — taxonomic assignment should be done on MOTU representative sequences, not individual reads
- Reference database is not formatted for OBITools (no obitax taxonomy); ecotag cannot resolve taxon names without it
- The marker region resolves species uniquely (no 100% multi-species matches) — then simple best-hit assignment suffices and MOTU splitting is unnecessary

## Inputs

- MOTU representative sequences (FASTA) produced by sumaclust 99% clustering, after removal of single-read MOTUs
- BOLD-derived COI reference database from github.com/metabarpark/Reference-databases, formatted for OBITools (obitax/ecoPCR-format)
- Per-sample read counts per MOTU (from the dereplication/clustering steps)

## Outputs

- Per-MOTU taxonomic assignment table (one row per MOTU) joining MOTU IDs, read counts per sample, and assigned taxonomy
- Flags for MOTUs with ambiguous 100%-identity matches to multiple BOLD species

## How to apply

Download the Metabarpark combined elasmobranch/non-elasmobranch COI reference database from github.com/metabarpark/Reference-databases and format it with OBITools (obitax/ecoPCR-format) so ecotag can resolve taxonomy strings. Run ecotag with the sumaclust 99%-similarity representative sequences as the query against the formatted combined reference database; ecotag assigns the most specific taxon consistent with all equal-best matches, which matters because 127 bp COI fragments can be 100% identical to several BOLD species. Parse the ecotag output and join MOTU IDs with per-sample read counts to build the assignment table. Validate that the table has one row per MOTU with populated taxonomy fields and that elasmobranch taxa contributing to the 21 reported shark species appear. Where a MOTU matches multiple species at 100% identity (e.g. Carcharhinus, Rhizoprionodon, Negaprion), do not force a single species call — flag it and split the MOTU into its Pacific and Caribbean components as the authors did.

## Related tools

- **ecotag** (Assigns each MOTU representative sequence the most specific consistent taxon against the formatted reference database) — github.com/metabarpark/Reference-databases
- **OBITools** (Suite used to format the reference database (obitax/ecoPCR-format) and to run ecotag) — github.com/metabarpark/Reference-databases
- **ecoPCR** (Used for the in silico PCR against EMBL-EBI release R117 that supplies the non-elasmobranch part of the combined reference database)

## Examples

```
ecotag -d combined_COI_refdb -R motu_representatives.fasta > ecotag_assignments.txt
```

## Evaluation signals

- Assignment table exists and contains exactly one row per MOTU with taxonomy fields populated
- Elasmobranch taxa identified in the study (contributing to the 21 shark species reported) are present in the assignments
- Every MOTU has at least one reference match; unmatched MOTUs are explicitly listed rather than silently dropped
- MOTUs with 100% identity to more than one BOLD species are flagged/split rather than given a single unambiguous species name
- Read counts per sample in the joined table sum to the read counts of the corresponding sumaclust MOTUs

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species; some MOTUs are 100% identical to multiple BOLD species, including species from different ocean regions, so MOTU-to-species mapping is not one-to-one
- No alternative primers amplifying a longer fragment were designed or tested; resolving closely related species is left to future work
- The combined reference database must be formatted for OBITools before ecotag can use it

## Evidence

- [discussion] Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75.: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75."
- [discussion] 100% multi-species BOLD matches: "the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database"
- [discussion] Bespoke reference database source: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [readme] README description of the reference databases: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
