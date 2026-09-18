---
name: metabarcoding-taxonomic-assignment
description: Use when you have MOTU representative sequences from a metabarcoding
  run (e.g. sumaclust 99% clusters, single-read MOTUs removed) and need to assign
  taxonomy against a curated reference database such as the BOLD-derived elasmobranch
  COI database.
when_to_use_negative:
- When reads have not yet been clustered into MOTUs — run the upstream pipeline (illuminapairedend
  >40 quality, obigrep 120–135 bp length filter, obiuniq, vsearch uchime-denovo, sumaclust
  99%) first.
- When the target marker/region needs resolution of closely related species pairs
  (e.g. Carcharhinus, Rhizoprionodon, Negaprion) and a longer COI fragment would be
  available — the 127 bp fragment is 100% identical across such species in BOLD, so
  ecotag cannot unambiguously separate them.
- When no suitable reference database exists for the taxon of interest — the study
  notes a bespoke database (BOLD COI sequences + ecoPCR in silico PCR non-targets)
  was required for assignment.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: ecotag
  role: Assigns the most specific consistent taxon to each MOTU representative sequence
    against the reference database
- name: OBITools
  role: Metabarcoding software suite that hosts ecotag and provides reference-database
    formatting utilities (obitax/ecoPCR-format)
- name: ecoPCR
  role: Used for in silico PCR against EMBL-EBI release R117 to retrieve non-elasmobranch
    COI sequences for the combined reference database
- name: sumaclust
  role: Upstream step that delimits MOTUs at a constant 99% similarity threshold and
    produces the representative sequences that ecotag assigns
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:c4da7920e787864368549a7b70ebfa9fc780445b4aebd00952cefd2eac0492d8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecoPCR
  - ecotag
  - sumaclust
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# metabarcoding-taxonomic-assignment

## Summary

Assigns a most-specific consistent taxon to each MOTU representative sequence (from 99%-similarity sumaclust clustering) using the OBITools ecotag algorithm against a BOLD-derived COI reference database. This converts clustered amplicon sequences into a per-MOTU taxonomy table, but the short 127 bp COI fragment may yield ambiguous one-to-many species matches.

## When to use

Use when you have MOTU representative sequences from a metabarcoding run (e.g. sumaclust 99% clusters, single-read MOTUs removed) and need to assign taxonomy against a curated reference database such as the BOLD-derived elasmobranch COI database.

## When NOT to use

- When reads have not yet been clustered into MOTUs — run the upstream pipeline (illuminapairedend >40 quality, obigrep 120–135 bp length filter, obiuniq, vsearch uchime-denovo, sumaclust 99%) first.
- When the target marker/region needs resolution of closely related species pairs (e.g. Carcharhinus, Rhizoprionodon, Negaprion) and a longer COI fragment would be available — the 127 bp fragment is 100% identical across such species in BOLD, so ecotag cannot unambiguously separate them.
- When no suitable reference database exists for the taxon of interest — the study notes a bespoke database (BOLD COI sequences + ecoPCR in silico PCR non-targets) was required for assignment.

## Inputs

- FASTA file of MOTU representative sequences from sumaclust 99%-similarity clustering (single-read MOTUs removed)
- BOLD-derived COI reference database (Metabarpark combined elasmobranch/non-elasmobranch), OBITools-formatted
- Per-sample MOTU x read-count table (e.g. from OBITools obistats/obiuniq output) to join with assignments

## Outputs

- ecotag output annotating each MOTU representative sequence with its most specific consistent taxon
- Per-MOTU taxonomic assignment table (MOTU ID, read counts per sample, assigned taxonomy)
- List of ambiguous MOTUs matching multiple reference species at 100% identity, flagged for splitting into Pacific/Caribbean components

## How to apply

First obtain the reference database: download the Metabarpark combined elasmobranch/non-elasmobranch COI reference database from github.com/metabarpark/Reference-databases and format it for OBITools (obitax/ecoPCR-format); the study built it by retrieving all COI elasmobranch sequences from BOLD (boldsystems.org) plus non-elasmobranch sequences found via in silico PCR with ecoPCR against EMBL-EBI release R117. Then run ecotag (OBITools suite) with the MOTU representative sequences from the sumaclust 99%-similarity clustering step, from which single-read MOTUs have been removed as a conservative contamination control, so that ecotag assigns the most specific consistent taxon each sequence shares with the reference pool. Aggregate the ecotag output into a per-MOTU table joining MOTU IDs, read counts per sample, and the assigned taxonomy. Expect that the short 127 bp COI fragment cannot always separate closely related species: some MOTUs match multiple species at 100% identity in BOLD (e.g. within Carcharhinus, Rhizoprionodon, Negaprion), in which case split those MOTUs into their ocean-region components (Pacific vs Caribbean) rather than forcing a one-to-one MOTU-to-species mapping. Validate that the assignment table has one populated taxonomy per MOTU and that elasmobranch taxa contributing to the 21 reported shark species are present.

## Related tools

- **ecotag** (Assigns the most specific consistent taxon to each MOTU representative sequence against the reference database)
- **OBITools** (Metabarcoding software suite that hosts ecotag and provides reference-database formatting utilities (obitax/ecoPCR-format))
- **ecoPCR** (Used for in silico PCR against EMBL-EBI release R117 to retrieve non-elasmobranch COI sequences for the combined reference database)
- **sumaclust** (Upstream step that delimits MOTUs at a constant 99% similarity threshold and produces the representative sequences that ecotag assigns)

## Examples

```
ecotag -R elasmobranch_refdb.fasta -r motu_representatives.fasta > motu_taxassignments.ecotag
```

## Evaluation signals

- Assignment table exists with one row per MOTU and taxonomy fields populated (no missing taxonomic fields).
- Elasmobranch taxa contributing to the 21 shark species reported in the study are present in the assignments.
- MOTUs with 100% identity to multiple BOLD species are identified and split into their Pacific and Caribbean components rather than collapsed to a single species name.
- Every representative sequence that was input to ecotag appears in the ecotag output (no MOTUs silently dropped).
- Reference database formatted successfully for OBITools (obitax/ecoPCR-format) and contains the combined elasmobranch and non-elasmobranch COI sequences.

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to individuals belonging to different species in BOLD, including species from different ocean regions.
- Some elasmobranch species present at sampling could not be detected due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered), so the reference-assignment pipeline reflects primer bias.
- A trade-off exists between taxonomic resolution and non-target amplification that needs balancing before informing conservation decisions; resolving closely related species would require designing alternative primers amplifying a longer gene fragment.
- Reference databases from the Metabarpark repository are provided in fasta format for Eukaryotic metabarcoding and must be formatted for OBITools before use with ecotag.

## Evidence

- [discussion] ecotag assignment of MOTU representatives: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75."
- [discussion] reference database availability: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [discussion] ambiguous MOTU-to-species matches: "the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database"
- [readme] reference database purpose: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
