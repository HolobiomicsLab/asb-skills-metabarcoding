---
name: silico-pcr-primer-scanning
description: Use when a metabarcoding pipeline needs a reference database covering
  both target taxa and non-target lineages amplifiable by a specific primer pair,
  and the target-taxon records (e.g. BOLD COI sequences) must be complemented by homologs
  from a broader archive.
when_to_use_negative:
- The input reads were amplified with a different primer pair or target a different
  fragment length than 127 bp / 120–135 bp, so the ecoPCR-extracted homologs would
  not match the amplicon size filter.
- The analysis does not need non-target lineage coverage (e.g. only target-taxon assignment
  is required and the curated BOLD subset alone suffices).
- The goal is resolving closely related species (Carcharhinus, Rhizoprionodon, Negaprion)
  — the 127 bp COI fragment is insufficient and alternative primers amplifying a longer
  fragment would be needed instead.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: ecoPCR
  role: Performs the in silico PCR against EMBL-EBI release R117 using the elasmobranch-specific
    COI primer pair to extract homologous non-elasmobranch sequences of the correct
    target size
- name: custom R script (BOLD retrieval)
  role: Retrieves all COI elasmobranch sequences available from BOLD, keeping those
    containing the 127 bp target fragment
  repo: http://github.com/metabarpark
- name: ecotag
  role: Consumes the combined reference database to assign taxonomy to MOTU representative
    sequences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:4ca08796dc6b1e448d910bae1d5bdaa6473e775345ee96c7570f45df752b8b1d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - custom R script (BOLD retrieval)
  - ecoPCR
  - ecotag
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# In silico PCR primer scanning for reference database extension

## Summary

Builds a taxonomically complete reference database by computationally amplifying sequences with a primer pair (here the elasmobranch-specific COI primers targeting a 127 bp fragment) against a large sequence archive (EMBL-EBI release R117), and merging those homologous non-target sequences with curated taxon-specific records (BOLD COI elasmobranch sequences).

## When to use

Use when a metabarcoding pipeline needs a reference database covering both target taxa and non-target lineages amplifiable by a specific primer pair, and the target-taxon records (e.g. BOLD COI sequences) must be complemented by homologs from a broader archive.

## When NOT to use

- The input reads were amplified with a different primer pair or target a different fragment length than 127 bp / 120–135 bp, so the ecoPCR-extracted homologs would not match the amplicon size filter.
- The analysis does not need non-target lineage coverage (e.g. only target-taxon assignment is required and the curated BOLD subset alone suffices).
- The goal is resolving closely related species (Carcharhinus, Rhizoprionodon, Negaprion) — the 127 bp COI fragment is insufficient and alternative primers amplifying a longer fragment would be needed instead.

## Inputs

- Primer pair sequences (elasmobranch-specific COI primers targeting a 127 bp fragment, Fields et al. 2015)
- Curated target-taxon sequence records: COI elasmobranch sequences from BOLD (boldsystems.org) retrieved via custom R script
- Large nucleotide archive: EMBL-EBI database release R117

## Outputs

- Combined reference database FASTA merging BOLD elasmobranch sequences and ecoPCR-extracted non-elasmobranch sequences
- Published reference database release (github.com/metabarpark/Reference-databases)

## How to apply

First retrieve all target-taxon sequences containing the amplicon from a curated database — here a custom R script fetched all COI elasmobranch sequences from BOLD (boldsystems.org), keeping only those containing the 127 bp target fragment. Then run ecoPCR in silico PCR with the same elasmobranch-specific COI primer pair against release R117 of the EMBL-EBI database, restricting output to fragments of the correct target size (120–135 bp, matching the obigrep length filter applied to the sequencing reads) to extract homologous non-elasmobranch sequences. Merge the BOLD target-taxon records and the ecoPCR-derived non-target records into a single combined FASTA reference database, subsequently used by ecotag for taxonomic assignment of the 99%-similarity sumaclust MOTUs. Validate the combined FASTA against a published release (github.com/metabarpark/Reference-databases) for sequence content and taxonomic coverage.

## Related tools

- **ecoPCR** (Performs the in silico PCR against EMBL-EBI release R117 using the elasmobranch-specific COI primer pair to extract homologous non-elasmobranch sequences of the correct target size)
- **custom R script (BOLD retrieval)** (Retrieves all COI elasmobranch sequences available from BOLD, keeping those containing the 127 bp target fragment) — http://github.com/metabarpark
- **ecotag** (Consumes the combined reference database to assign taxonomy to MOTU representative sequences)

## Evaluation signals

- The combined FASTA exists, is well-formed, and contains both the BOLD-derived elasmobranch records and the ecoPCR-derived non-elasmobranch records.
- All retrieved sequences contain the 127 bp target fragment / fall within the 120–135 bp target-size window.
- The sequence content and taxonomic coverage of the combined database are comparable with the published FASTA release at github.com/metabarpark/Reference-databases.
- ecotag assignment against the combined database correctly assigns known elasmobranch MOTUs (e.g. the ≥21 shark species detected) while non-target sequences provide outgroup assignments.
- The custom R scripts used for retrieval are publicly available from http://github.com/metabarpark, allowing reproducibility checks.

## Limitations

- The 127 bp COI fragment gives insufficient taxonomic resolution to separate closely related species; some MOTUs are 100% identical to individuals belonging to different species in the BOLD database, including species from different ocean regions.
- A trade-off exists between taxonomic resolution and non-target amplification that must be balanced before informing conservation decisions.
- Certain elasmobranch species present in the environment could not be detected with the selected primer set due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered).

## Evidence

- [discussion] In silico PCR scan with target-size filtering: "an in silico PCR was performed against release R117 of the EMBL-EBI database using ecoPCR78"
- [discussion] BOLD target-taxon retrieval: "We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database"
- [discussion] Target-size criterion: "a length filter (obigrep) was applied to the aligned reads (120–135 bp) in order to select only the fragments with the correct target size."
- [discussion] Published combined database release: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [readme] Repository purpose: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
