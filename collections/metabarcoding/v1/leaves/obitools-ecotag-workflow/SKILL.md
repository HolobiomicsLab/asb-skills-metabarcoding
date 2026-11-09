---
name: obitools-ecotag-workflow
description: Use when MOTU representative sequences (e.g. from sumaclust 99% clustering)
  need taxonomic assignment against a curated COI reference database to label each
  MOTU with the most specific consistent taxon.
when_to_use_negative:
- Input is raw reads or aligned reads that have not yet been dereplicated, chimera-filtered
  (uchime-denovo in vsearch), and clustered into MOTUs at 99% similarity — run the
  upstream OBITools pipeline first.
- The reference database does not cover the target group, or the 127 bp COI fragment
  is expected to be 100% identical among closely related species (e.g. Carcharhinus,
  Rhizoprionodon, Negaprion) and unambiguous species-level resolution is required.
- A longer marker or alternative primers are needed to separate closely related species;
  the article states this resolution requires new primers amplifying a longer fragment,
  not ecotag.
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_3316
tools:
- name: OBITools
  role: Metabarcoding software suite providing ecotag and reference-database formatting
    (obitax/ecoPCR-format) used for taxonomic assignment
  repo: github.com/metabarpark/Reference-databases
- name: ecotag
  role: Assigns the most specific consistent taxon to each MOTU representative sequence
    against the combined COI reference database
- name: sumaclust
  role: Upstream MOTU delimitation at 99% similarity that produces the representative
    sequences fed into ecotag
- name: ecoPCR
  role: In silico PCR against EMBL-EBI release R117 supplying non-elasmobranch sequences
    for the combined reference database
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:ad24675c731f1ea0d6af5fe3553fe36bf5100a28bdbe61bc40b5db54388ce61c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecoPCR
  - ecotag
  - sumaclust
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_3316
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# obitools-ecotag-workflow

## Summary

Assigns MOTU representative sequences to taxa using the OBITools ecotag algorithm against a bespoke COI reference database (BOLD-derived elasmobranch sequences plus in silico PCR non-elasmobranch sequences), producing a per-MOTU taxonomic assignment table.

## When to use

Use when MOTU representative sequences (e.g. from sumaclust 99% clustering) need taxonomic assignment against a curated COI reference database to label each MOTU with the most specific consistent taxon.

## When NOT to use

- Input is raw reads or aligned reads that have not yet been dereplicated, chimera-filtered (uchime-denovo in vsearch), and clustered into MOTUs at 99% similarity — run the upstream OBITools pipeline first.
- The reference database does not cover the target group, or the 127 bp COI fragment is expected to be 100% identical among closely related species (e.g. Carcharhinus, Rhizoprionodon, Negaprion) and unambiguous species-level resolution is required.
- A longer marker or alternative primers are needed to separate closely related species; the article states this resolution requires new primers amplifying a longer fragment, not ecotag.

## Inputs

- MOTU representative sequences (FASTA) from sumaclust 99% clustering, with single-read MOTUs removed
- Combined elasmobranch/non-elasmobranch COI reference database from github.com/metabarpark/Reference-databases, formatted for OBITools
- Per-sample MOTU read-count table for joining assignments to abundances

## Outputs

- ecotag output with taxonomic assignment for each MOTU representative sequence
- Per-MOTU taxonomic assignment table (MOTU ID, read counts per sample, assigned taxonomy)

## How to apply

Download the Metabarpark combined elasmobranch/non-elasmobranch COI reference database from github.com/metabarpark/Reference-databases and format it for OBITools (obitax/ecoPCR-format). Load the MOTU representative sequences produced by the sumaclust 99%-similarity clustering step, after single-read MOTUs have been removed. Run ecotag with the representative sequences against the combined reference database so each MOTU receives the most specific consistent taxon supported by its reference matches. Aggregate the ecotag output into a per-MOTU table joining MOTU IDs, per-sample read counts, and assigned taxonomy. Validate that every row has populated taxonomy fields and that the elasmobranch taxa contributing to the 21 reported shark species appear in the assignments; note that some MOTUs match multiple species at 100% identity and were split into Pacific and Caribbean components.

## Related tools

- **OBITools** (Metabarcoding software suite providing ecotag and reference-database formatting (obitax/ecoPCR-format) used for taxonomic assignment) — github.com/metabarpark/Reference-databases
- **ecotag** (Assigns the most specific consistent taxon to each MOTU representative sequence against the combined COI reference database)
- **sumaclust** (Upstream MOTU delimitation at 99% similarity that produces the representative sequences fed into ecotag)
- **ecoPCR** (In silico PCR against EMBL-EBI release R117 supplying non-elasmobranch sequences for the combined reference database)

## Examples

```
ecotag -R elasmobranch_COI_ref.fasta -r motu_representatives.fasta --assignment-cutoff=0.0 > ecotag_assignments.fasta
```

## Evaluation signals

- Assignment table exists and contains one row per MOTU with taxonomy fields populated.
- Elasmobranch taxa identified in the study (contributing to the 21 shark species reported) are present in the ecotag assignments.
- MOTUs with 100% identity matches to more than one BOLD species are flagged and handled (e.g. split into Pacific and Caribbean components) rather than force-assigned to one species.
- Representative sequences entering ecotag correspond exactly to the sumaclust 99% MOTU set after single-read MOTU removal.
- Reference database successfully formatted for OBITools (obitax/ecoPCR-format) with taxonomy records attached before ecotag runs.

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple BOLD species, sometimes from different ocean regions.
- Resolving closely related species requires alternative primers amplifying a longer gene fragment, which were not designed or tested in this study.
- Some elasmobranch species present at sampling could not be detected due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered).
- A trade-off exists between taxonomic resolution and non-target amplification that must be balanced before informing conservation decisions.

## Evidence

- [discussion] Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm"
- [discussion] MOTUs delimited at 99% similarity: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] Bespoke reference database from BOLD: "We built a bespoke elasmobranch reference database using a custom R script for retrieving all COI elasmobranch sequences available from the BOLD database"
- [discussion] 100% identity to multiple species: "the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database"
- [readme] Combined reference database availability: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
