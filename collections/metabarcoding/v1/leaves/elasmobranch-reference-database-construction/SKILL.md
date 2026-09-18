---
name: elasmobranch-reference-database-construction
description: Use when assigning 127-bp COI eDNA metabarcoding reads (amplified with
  Fields et al. elasmobranch-specific primers) to shark/ray taxa and no suitable combined
  elasmobranch+non-target reference database exists yet.
when_to_use_negative:
- The taxonomic assignment target is not short COI eDNA reads from elasmobranch-specific
  primers (e.g. full-length COI barcodes or other marker genes) — retrieve or reuse
  a marker-appropriate reference database instead.
- The published combined database at github.com/metabarpark/Reference-databases already
  matches your primer pair and target fragment; downloading it is preferable to rebuilding.
- You need to resolve closely related species (e.g. Carcharhinus, Rhizoprionodon,
  Negaprion) — the 127-bp fragment leaves some MOTUs 100% identical to multiple species
  in BOLD, so this database cannot separate them; longer-amplicon primers would be
  required.
edam_operation: http://edamontology.org/operation_0346
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0084
tools:
- name: OBITools
  role: metabarcoding suite whose ecotag algorithm consumes the combined reference
    database for taxonomic assignment of MOTU representative sequences
  repo: github.com/metabarpark/Reference-databases
- name: ecoPCR
  role: in silico PCR against EMBL-EBI release R117 using the elasmobranch-specific
    COI primer pair to extract non-elasmobranch homologous sequences of 120–135 bp
- name: custom R script
  role: retrieves all COI elasmobranch sequences from the BOLD database, keeping sequences
    containing the 127 bp target fragment
  repo: github.com/metabarpark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:e101eff44afdebc60aefa0ee29b1b467269404fc4861d9f40e2ce29e79d1e4df
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabarpark/Reference-databases
  tools:
  - OBITools
  - custom R script
  - ecoPCR
  edam_operation: http://edamontology.org/operation_0346
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0084
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# elasmobranch-reference-database-construction

## Summary

Assembles a bespoke COI reference database for taxonomic assignment of short (127-bp) elasmobranch eDNA reads by merging all BOLD elasmobranch COI sequences (retrieved via custom R script) with ecoPCR-derived non-elasmobranch sequences from EMBL-EBI release R117, for use with OBITools' ecotag.

## When to use

Use when assigning 127-bp COI eDNA metabarcoding reads (amplified with Fields et al. elasmobranch-specific primers) to shark/ray taxa and no suitable combined elasmobranch+non-target reference database exists yet.

## When NOT to use

- The taxonomic assignment target is not short COI eDNA reads from elasmobranch-specific primers (e.g. full-length COI barcodes or other marker genes) — retrieve or reuse a marker-appropriate reference database instead.
- The published combined database at github.com/metabarpark/Reference-databases already matches your primer pair and target fragment; downloading it is preferable to rebuilding.
- You need to resolve closely related species (e.g. Carcharhinus, Rhizoprionodon, Negaprion) — the 127-bp fragment leaves some MOTUs 100% identical to multiple species in BOLD, so this database cannot separate them; longer-amplicon primers would be required.

## Inputs

- BOLD database (boldsystems.org) COI elasmobranch sequence records (filtered to those containing the 127 bp target fragment)
- EMBL-EBI database release R117 (for ecoPCR in silico PCR)
- Elasmobranch-specific COI primer pair sequences (Fields et al. primer set, 127 bp amplicon)

## Outputs

- Combined FASTA reference database file merging BOLD elasmobranch COI sequences and ecoPCR-derived non-elasmobranch sequences (120–135 bp)

## How to apply

First, run a custom R script to retrieve all COI elasmobranch sequences available from the BOLD database (boldsystems.org), keeping only sequences that contain the 127 bp target fragment. Second, run ecoPCR in silico PCR against release R117 of the EMBL-EBI database using the elasmobranch-specific COI primer pair to extract homologous non-elasmobranch sequences with correct target size (120–135 bp). Third, merge the BOLD elasmobranch sequences and ecoPCR-derived non-elasmobranch sequences into a single combined FASTA reference database. The resulting database is then consumed by the ecotag algorithm within the OBITools suite for taxonomic assignment of MOTU representative sequences; validate the output by checking the combined FASTA exists, is well-formed, and matches the published release at github.com/metabarpark/Reference-databases in sequence content and taxonomic coverage.

## Related tools

- **OBITools** (metabarcoding suite whose ecotag algorithm consumes the combined reference database for taxonomic assignment of MOTU representative sequences) — github.com/metabarpark/Reference-databases
- **ecoPCR** (in silico PCR against EMBL-EBI release R117 using the elasmobranch-specific COI primer pair to extract non-elasmobranch homologous sequences of 120–135 bp)
- **custom R script** (retrieves all COI elasmobranch sequences from the BOLD database, keeping sequences containing the 127 bp target fragment) — github.com/metabarpark

## Examples

```
Rscript retrieve_BOLD_elasmobranch_COI.R && ecoPCR -l 120 -L 135 primerF primerA embl_R117.embl > non_elasmobranch.ecopcr && cat bold_elasmobranch.fasta non_elasmobranch.fasta > elasmobranch_COI_127bp.fasta
```

## Evaluation signals

- The combined FASTA reference database file exists and is well-formed (parseable FASTA).
- Sequence content and taxonomic coverage of the rebuilt database are comparable with the published FASTA release at github.com/metabarpark/Reference-databases.
- Elasmobranch component contains all COI elasmobranch sequences from BOLD that include the 127 bp target fragment.
- Non-elasmobranch component consists of ecoPCR products of correct target size (120–135 bp) from EMBL-EBI release R117, amplified with the elasmobranch-specific primer pair.

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species; some MOTUs are 100% identical to individuals of different species in BOLD, including species from different ocean regions.
- Certain elasmobranch species present at sampling cannot be detected with this primer pair due to primer mismatches (e.g. Ginglymostoma cirratum was observed visually but yielded no reads).
- The trade-off between taxonomic resolution and non-target amplification needs to be well balanced before applying the approach to conservation decisions.
- The reference database release is version-locked to BOLD content and EMBL-EBI release R117; later database releases may change assignments.

## Evidence

- [methods] We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database: "We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database"
- [methods] an in silico PCR was performed against release R117 of the EMBL-EBI database using ecoPCR: "an in silico PCR was performed against release R117 of the EMBL-EBI database using ecoPCR"
- [methods] Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm"
- [methods] This combined reference database is available from http://github.com/metabarpark/Reference-databases.: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [discussion] the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database: "the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database"
