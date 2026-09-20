---
name: fasta-reference-database-curation
description: Use when assigning taxonomy to short (120–135 bp) COI eDNA amplicon reads
  from elasmobranch samples and no ready-made reference covering both target elasmobranchs
  and non-target homologs exists.
when_to_use_negative:
- The marker is not the 127 bp COI fragment amplified by the elasmobranch-specific
  primer pair — an in silico PCR-extracted database matched to a different primer
  pair or locus is inappropriate.
- Species-level resolution among closely related elasmobranchs (e.g. Carcharhinus,
  Rhizoprionodon, Negaprion) is required; some MOTUs are 100% identical to multiple
  BOLD species, and the authors note longer-fragment primers would be needed.
- A published release comparable in sequence content and taxonomic coverage (github.com/metabarpark/Reference-databases)
  already satisfies the need — reuse it rather than rebuilding.
tools:
- name: custom R script
  role: Retrieves all COI elasmobranch sequences available from the BOLD database,
    keeping sequences containing the 127 bp target fragment
  repo: http://github.com/metabarpark
- name: ecoPCR
  role: Performs in silico PCR against EMBL-EBI release R117 with the elasmobranch
    COI primer pair to extract non-elasmobranch homologous sequences of the correct
    target size
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:d07a23b33c325ccb73cd8378ea2151334025b3e90039458b7e70040ba64bf061
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - custom R script
  - ecoPCR
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# fasta-reference-database-curation

## Summary

Assembles a bespoke elasmobranch COI reference FASTA by merging BOLD-derived elasmobranch sequences (containing the 127 bp target fragment) with ecoPCR-extracted non-elasmobranch homologs, for use with ecotag taxonomic assignment in eDNA metabarcoding.

## When to use

Use when assigning taxonomy to short (120–135 bp) COI eDNA amplicon reads from elasmobranch samples and no ready-made reference covering both target elasmobranchs and non-target homologs exists.

## When NOT to use

- The marker is not the 127 bp COI fragment amplified by the elasmobranch-specific primer pair — an in silico PCR-extracted database matched to a different primer pair or locus is inappropriate.
- Species-level resolution among closely related elasmobranchs (e.g. Carcharhinus, Rhizoprionodon, Negaprion) is required; some MOTUs are 100% identical to multiple BOLD species, and the authors note longer-fragment primers would be needed.
- A published release comparable in sequence content and taxonomic coverage (github.com/metabarpark/Reference-databases) already satisfies the need — reuse it rather than rebuilding.

## Inputs

- BOLD database (boldsystems.org) COI elasmobranch sequence records
- EMBL-EBI database release R117
- Elasmobranch-specific COI primer pair sequences (Fields et al. 2015)
- Custom R retrieval script (github.com/metabarpark)

## Outputs

- Combined FASTA reference database merging BOLD elasmobranch COI sequences and ecoPCR non-elasmobranch sequences (120–135 bp amplicons)

## How to apply

First, retrieve all COI elasmobranch sequences from the BOLD database (boldsystems.org) using a custom R script (scripts published at github.com/metabarpark), keeping only sequences that contain the 127 bp target fragment. Second, run ecoPCR in silico PCR against release R117 of the EMBL-EBI database using the elasmobranch-specific COI primer pair to extract homologous non-elasmobranch sequences whose amplicon size falls within the 120–135 bp target window. Third, merge the BOLD elasmobranch sequences and the ecoPCR-derived non-elasmobranch sequences into a single combined FASTA reference database. The rationale is that ecotag assignment needs both target-group sequences for species-level labels and off-target homologs so that non-elasmobranch reads are not misassigned. The published combined database is deposited at github.com/metabarpark/Reference-databases for direct comparison.

## Related tools

- **custom R script** (Retrieves all COI elasmobranch sequences available from the BOLD database, keeping sequences containing the 127 bp target fragment) — http://github.com/metabarpark
- **ecoPCR** (Performs in silico PCR against EMBL-EBI release R117 with the elasmobranch COI primer pair to extract non-elasmobranch homologous sequences of the correct target size)

## Evaluation signals

- The combined reference database FASTA exists and is well-formed.
- Sequence content and taxonomic coverage are comparable with the published FASTA release at github.com/metabarpark/Reference-databases.
- BOLD-derived sequences retained each contain the 127 bp target fragment.
- ecoPCR-derived non-elasmobranch sequences fall within the 120–135 bp amplicon size window.

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species; some MOTUs are 100% identical to individuals belonging to different species in BOLD, including species from different ocean regions.
- Resolving closely related species would require alternative primers amplifying a longer gene fragment, which was not designed or tested in this study.
- The trade-off between taxonomic resolution and non-target amplification needs to be well balanced before the database supports conservation decisions.

## Evidence

- [discussion] We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database: "We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database"
- [discussion] an in silico PCR was performed against release R117 of the EMBL-EBI database using ecoPCR: "an in silico PCR was performed against release R117 of the EMBL-EBI database using ecoPCR78"
- [discussion] This combined reference database is available from github.com/metabarpark/Reference-databases: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [discussion] some MOTUs are 100% identical to individuals belonging to different species in the BOLD database: "the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database"
- [readme] If you use the reference database for COI elasmobranch sequences using Fields et al. (2015) primers, please cite: "If you use the reference database for COI elasmobranch sequences using Fields et al. (2015) primers, please cite:"
