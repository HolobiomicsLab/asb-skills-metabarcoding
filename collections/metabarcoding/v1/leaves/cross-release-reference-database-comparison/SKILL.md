---
name: cross-release-reference-database-comparison
description: Use when building or validating a marker-specific eDNA reference database
  (e.g. for a 127-bp COI metabarcoding assay) and you must confirm your combined fasta
  matches a published reference release in sequence content and taxonomic coverage.
when_to_use_negative:
- Your input is already a MOTU/feature table or reads needing taxonomic assignment
  — use ecotag against the finished database instead of rebuilding it
- Your metabarcoding marker is not the 127 bp elasmobranch COI fragment (e.g. a longer
  COI fragment or 12S) — the BOLD fragment filter and 120–135 bp size window are specific
  to this primer pair
- You only need non-elasmobranch off-target sequences and already have a validated
  combined release from github.com/metabarpark/Reference-databases to download
edam_operation: http://edamontology.org/operation_0361
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: custom R script
  role: Retrieves all COI elasmobranch sequences from the BOLD database, keeping sequences
    that contain the 127 bp target fragment; scripts are publicly available
  repo: http://github.com/metabarpark
- name: ecoPCR
  role: Performs in silico PCR against EMBL-EBI release R117 with the elasmobranch-specific
    COI primer pair to extract homologous non-elasmobranch sequences of the correct
    target size
- name: OBITools
  role: Metabarcoding suite in which the combined database is consumed for taxonomic
    assignment (ecotag) of the 127-bp COI MOTU representative sequences
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:556e71e6881487665469148b0a367b62369140cee0ea9205465acc8b30a5c812
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - custom R script
  - ecoPCR
  edam_operation: http://edamontology.org/operation_0361
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-release-reference-database-comparison

## Summary

Assembles a bespoke elasmobranch COI reference database (BOLD elasmobranch sequences plus ecoPCR-extracted non-elasmobranch homologs) and validates it by checking sequence content and taxonomic coverage against the published fasta release deposited at github.com/metabarpark/Reference-databases.

## When to use

Use when building or validating a marker-specific eDNA reference database (e.g. for a 127-bp COI metabarcoding assay) and you must confirm your combined fasta matches a published reference release in sequence content and taxonomic coverage.

## When NOT to use

- Your input is already a MOTU/feature table or reads needing taxonomic assignment — use ecotag against the finished database instead of rebuilding it
- Your metabarcoding marker is not the 127 bp elasmobranch COI fragment (e.g. a longer COI fragment or 12S) — the BOLD fragment filter and 120–135 bp size window are specific to this primer pair
- You only need non-elasmobranch off-target sequences and already have a validated combined release from github.com/metabarpark/Reference-databases to download

## Inputs

- All COI elasmobranch sequences available from the BOLD database (boldsystems.org), filtered to entries containing the 127 bp target fragment
- Release R117 of the EMBL-EBI database (for in silico PCR with the elasmobranch-specific COI primer pair)
- Elasmobranch-specific COI primer pair (Fields et al. 2015) targeting a 127 bp mitochondrial COI stretch
- Published reference fasta release from github.com/metabarpark/Reference-databases (comparison/validation target)

## Outputs

- Single combined fasta reference database file merging BOLD elasmobranch sequences and ecoPCR-derived non-elasmobranch sequences (120–135 bp)
- Validation report confirming the combined fasta is well-formed and comparable (sequence content and taxonomic coverage) with the github.com/metabarpark/Reference-databases release

## How to apply

Retrieve all COI elasmobranch sequences from the BOLD database (boldsystems.org) with a custom R script, retaining only sequences that contain the 127 bp target fragment of the elasmobranch-specific COI primer pair. Then run ecoPCR in silico PCR against release R117 of the EMBL-EBI database to extract homologous non-elasmobranch sequences of the correct target size (120–135 bp, matching the obigrep length filter used downstream). Merge the BOLD elasmobranch fasta and the ecoPCR-derived non-elasmobranch fasta into a single combined reference database file. Validate by verifying the combined fasta exists, is well-formed, and is comparable in sequence content and taxonomic coverage with the published fasta release at github.com/metabarpark/Reference-databases; this combined database then feeds taxonomic assignment via the ecotag algorithm (OBITools).

## Related tools

- **custom R script** (Retrieves all COI elasmobranch sequences from the BOLD database, keeping sequences that contain the 127 bp target fragment; scripts are publicly available) — http://github.com/metabarpark
- **ecoPCR** (Performs in silico PCR against EMBL-EBI release R117 with the elasmobranch-specific COI primer pair to extract homologous non-elasmobranch sequences of the correct target size)
- **OBITools** (Metabarcoding suite in which the combined database is consumed for taxonomic assignment (ecotag) of the 127-bp COI MOTU representative sequences)

## Examples

```
ecoPCR -m 120 -M 135 elasmobranch_COI_F elasmobranch_COI_R embl_R117 > non_elasmobranch_120-135bp.ecopcr
```

## Evaluation signals

- The combined reference database fasta exists and is well-formed (parses as a valid fasta file)
- Sequence content and taxonomic coverage of the rebuilt database are comparable with the published fasta release at github.com/metabarpark/Reference-databases
- All retained BOLD elasmobranch sequences contain the 127 bp target fragment
- ecoPCR-extracted non-elasmobranch sequences fall within the 120–135 bp target size window used for downstream length filtering
- BOLD elasmobranch and ecoPCR non-elasmobranch components are both present in the merged single fasta file

## Limitations

- The 127 bp COI fragment lacks taxonomic resolution to separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to individuals belonging to different species in the BOLD database, including species from different ocean regions
- Certain elasmobranch species present at sampling (e.g. Ginglymostoma cirratum, observed visually) could not be detected due to primer mismatches, so database coverage does not guarantee detection
- The trade-off between taxonomic resolution and non-target amplification is not fully resolved and must be well balanced before applying the approach to inform conservation decisions
- The Reference-databases README states the databases are intended for Eukaryotic metabarcoding of marine benthic communities and asks citation of Wangensteen & Turon (2016) and Bakker et al. (2017) when using the COI elasmobranch database

## Evidence

- [discussion] Bespoke elasmobranch reference database construction: "We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database"
- [discussion] In silico PCR against EMBL-EBI R117: "an in silico PCR was performed against release R117 of the EMBL-EBI database using ecoPCR"
- [discussion] Published release location for comparison: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [discussion] Downstream taxonomic assignment with the database: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm"
- [readme] README scope of the reference databases: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
