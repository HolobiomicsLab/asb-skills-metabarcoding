---
name: bold-c-oi-sequence-retrieval-replace
description: Use when building a reference database for taxonomic assignment of short
  COI metabarcoding reads (120–135 bp target fragment) where public curated barcoding
  sequences (BOLD) must be supplemented with non-target sequences to quantify/contain
  non-elasmobranch amplification.
when_to_use_negative:
- The target marker is not the 127 bp mitochondrial COI fragment amplified by the
  elasmobranch-specific primer pair (e.g. a longer fragment designed to resolve closely
  related Carcharhinus/Rhizoprionodon/Negaprion species, which the authors left to
  future work).
- A general-purpose public reference database release already covers the required
  taxa and no bespoke merging with non-target sequences is needed.
- The goal is assembling a reference database for non-elasmobranch eukaryotic groups;
  this skill is specific to elasmobranch COI plus ecoPCR-derived non-elasmobranch
  homologs.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: custom R script
  role: Retrieves all COI elasmobranch sequences available from the BOLD database,
    keeping sequences containing the 127 bp target fragment
  repo: http://github.com/metabarpark
- name: ecoPCR
  role: Performs in silico PCR against EMBL-EBI release R117 to extract homologous
    non-elasmobranch sequences of the correct target size (120–135 bp) for the combined
    database
- name: OBITools
  role: Metabarcoding suite whose ecotag algorithm consumes the combined fasta reference
    database for taxonomic assignment of MOTU representative sequences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:4748ecaded72b662a8b9ca95b585cd527d83f8ce4765be4546a0034ccc658be2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - custom R script
  - ecoPCR
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# bold-c oi-sequence-retrieval".replace("

## Summary

Assembles a bespoke elasmobranch COI reference database for taxonomic assignment of 127-bp eDNA metabarcoding reads by retrieving all COI elasmobranch sequences from BOLD (boldsystems.org) with a custom R script and merging them with non-elasmobranch homologs extracted via ecoPCR in silico PCR.

## When to use

Use when building a reference database for taxonomic assignment of short COI metabarcoding reads (120–135 bp target fragment) where public curated barcoding sequences (BOLD) must be supplemented with non-target sequences to quantify/contain non-elasmobranch amplification.

## When NOT to use

- The target marker is not the 127 bp mitochondrial COI fragment amplified by the elasmobranch-specific primer pair (e.g. a longer fragment designed to resolve closely related Carcharhinus/Rhizoprionodon/Negaprion species, which the authors left to future work).
- A general-purpose public reference database release already covers the required taxa and no bespoke merging with non-target sequences is needed.
- The goal is assembling a reference database for non-elasmobranch eukaryotic groups; this skill is specific to elasmobranch COI plus ecoPCR-derived non-elasmobranch homologs.

## Inputs

- BOLD database (boldsystems.org) COI elasmobranch sequence records
- EMBL-EBI database release R117
- Elasmobranch-specific COI primer pair targeting a 127 bp fragment (Fields et al. 2015 primers)

## Outputs

- Combined fasta reference database file of elasmobranch COI sequences plus ecoPCR-derived non-elasmobranch homologs (120–135 bp)
- Fasta reference database published at github.com/metabarpark/Reference-databases

## How to apply

First, run a custom R script (publicly available from github.com/metabarpark) against the BOLD database (boldsystems.org) to retrieve all available COI elasmobranch sequences, retaining only those that contain the 127 bp target fragment amplified by the elasmobranch-specific primer pair. Second, run ecoPCR in silico PCR against release R117 of the EMBL-EBI database with the same primer pair to extract homologous non-elasmobranch sequences of the correct target size (120–135 bp), mirroring the expected amplicon range used in the obigrep length filter. Third, merge the BOLD elasmobranch sequences and the ecoPCR-derived non-elasmobranch sequences into a single combined fasta reference database, which is then consumed downstream by the OBITools ecotag algorithm for MOTU taxonomic assignment. The combined database should match the published fasta release deposited at github.com/metabarpark/Reference-databases.

## Related tools

- **custom R script** (Retrieves all COI elasmobranch sequences available from the BOLD database, keeping sequences containing the 127 bp target fragment) — http://github.com/metabarpark
- **ecoPCR** (Performs in silico PCR against EMBL-EBI release R117 to extract homologous non-elasmobranch sequences of the correct target size (120–135 bp) for the combined database)
- **OBITools** (Metabarcoding suite whose ecotag algorithm consumes the combined fasta reference database for taxonomic assignment of MOTU representative sequences)

## Evaluation signals

- The combined reference database fasta exists and is well-formed (parseable fasta).
- The combined database is comparable in sequence content and taxonomic coverage with the published fasta release deposited at github.com/metabarpark/Reference-databases.
- All BOLD-derived elasmobranch sequences retained in the database contain the 127 bp target fragment.
- All ecoPCR-derived non-elasmobranch sequences fall within the correct target size range (120–135 bp).
- The in silico PCR was run against release R117 of the EMBL-EBI database using the same elasmobranch-specific COI primer pair as the wet-lab PCR.

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to individuals of different species in BOLD, including species from different ocean regions.
- Certain elasmobranch species present in the environment could not be detected with the selected primer set due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered), which is a caveat inherited by the reference database's utility.
- A trade-off between taxonomic resolution and non-target amplification was not resolved and needs to be balanced before informing conservation decisions; the authors note alternative primers amplifying a longer fragment will be essential to resolve closely related species.

## Evidence

- [discussion] custom R script retrieving BOLD elasmobranch COI sequences: "We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database"
- [discussion] ecoPCR in silico PCR against EMBL-EBI R117: "an in silico PCR was performed against release R117 of the EMBL-EBI database using ecoPCR78"
- [discussion] availability of the combined reference database: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [readme] README citation for the COI elasmobranch reference database: "If you use the reference database for COI elasmobranch sequences using Fields et al. (2015) primers, please cite: Bakker et al. (2017) (in preparation)."
- [discussion] downstream use in taxonomic assignment: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75."
