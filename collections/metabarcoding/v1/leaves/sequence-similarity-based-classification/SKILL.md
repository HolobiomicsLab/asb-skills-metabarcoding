---
name: sequence-similarity-based-classification
description: Use when MOTU representative sequences from 99% sumaclust clustering
  of COI eDNA reads must be assigned to taxa against a curated reference database,
  and one-to-one MOTU-to-species mapping must be checked.
when_to_use_negative:
- When reads are still raw or unclustered — the skill requires post-clustering representative
  sequences; run sumaclust at 99% and remove single-read MOTUs first
- When the target marker is not the 127 bp mitochondrial COI fragment amplified by
  the Fields et al. (2015) elasmobranch primers — the reference database and identity-based
  assignment logic are COI-specific
- When unambiguous species-level resolution of closely related genera (Carcharhinus,
  Rhizoprionodon, Negaprion) is required — this ecotag approach cannot separate species
  that are 100% identical in BOLD; longer-fragment primers are needed
edam_operation: http://edamontology.org/operation_0291
edam_topics:
- http://edamontology.org/topic_0654
- http://edamontology.org/topic_0080
- http://edamontology.org/topic_3174
tools:
- name: ecotag
  role: Assigns the most specific consistent taxon to each MOTU representative sequence
    against the combined COI reference database
  repo: github.com/metabarpark/Reference-databases
- name: OBITools
  role: Metabarcoding software suite providing ecotag and the taxonomy/database formatting
    utilities (obitax, ecoPCR-format)
- name: sumaclust
  role: Upstream producer of the 99%-similarity MOTU representative sequences that
    serve as input to ecotag
- name: ecoPCR
  role: In silico PCR used to build the non-elasmobranch part of the reference database
    from EMBL-EBI release R117
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:114e705176332fcf96836fae29d75a6615a7fc7366665dbebc354f59eb6936f3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabarpark/Reference-databases
  tools:
  - OBITools
  - ecoPCR
  - ecotag
  - sumaclust
  edam_operation: http://edamontology.org/operation_0291
  edam_topics:
  - http://edamontology.org/topic_0654
  - http://edamontology.org/topic_0080
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# sequence-similarity-based-classification

## Summary

Assign each 99%-similarity MOTU representative sequence to the most specific consistent taxon using OBITools' ecotag against a BOLD-derived COI reference database (combined elasmobranch/non-elasmobranch, formatted with obitax/ecoPCR-format), producing a per-MOTU taxonomy table.

## When to use

Use when MOTU representative sequences from 99% sumaclust clustering of COI eDNA reads must be assigned to taxa against a curated reference database, and one-to-one MOTU-to-species mapping must be checked.

## When NOT to use

- When reads are still raw or unclustered — the skill requires post-clustering representative sequences; run sumaclust at 99% and remove single-read MOTUs first
- When the target marker is not the 127 bp mitochondrial COI fragment amplified by the Fields et al. (2015) elasmobranch primers — the reference database and identity-based assignment logic are COI-specific
- When unambiguous species-level resolution of closely related genera (Carcharhinus, Rhizoprionodon, Negaprion) is required — this ecotag approach cannot separate species that are 100% identical in BOLD; longer-fragment primers are needed

## Inputs

- MOTU representative sequences (FASTA) from sumaclust 99% clustering, with single-read MOTUs removed
- Combined elasmobranch/non-elasmobranch COI reference database (FASTA from github.com/metabarpark/Reference-databases), formatted for OBITools (obitax/ecoPCR-format)
- Per-sample read counts per MOTU for building the assignment table

## Outputs

- ecotag assignment output per representative sequence
- Per-MOTU taxonomic assignment table joining MOTU IDs, read counts per sample, and assigned taxonomy
- List of ambiguous MOTUs matched at 100% identity to multiple BOLD species (split into Pacific/Caribbean components)

## How to apply

Download the Metabarpark combined elasmobranch/non-elasmobranch COI reference database from github.com/metabarpark/Reference-databases and format it for OBITools using obitax/ecoPCR-format. Take the MOTU representative sequences produced by the sumaclust 99%-similarity clustering step, from which single-read MOTUs have already been removed. Run ecotag (OBITools suite) with the representative sequences against the combined reference database so that each MOTU receives the most specific consistent taxon shared by all matching references. Aggregate the ecotag output into a per-MOTU table joining MOTU IDs, read counts per sample, and assigned taxonomy. Critically inspect assignments: because the 127 bp COI fragment gives limited resolution, some MOTUs matched at 100% sequence identity to more than one BOLD species; such ambiguous MOTUs (e.g. within Carcharhinus, Rhizoprionodon, Negaprion) were split into their Pacific and Caribbean components rather than forced to a single species.

## Related tools

- **ecotag** (Assigns the most specific consistent taxon to each MOTU representative sequence against the combined COI reference database) — github.com/metabarpark/Reference-databases
- **OBITools** (Metabarcoding software suite providing ecotag and the taxonomy/database formatting utilities (obitax, ecoPCR-format))
- **sumaclust** (Upstream producer of the 99%-similarity MOTU representative sequences that serve as input to ecotag)
- **ecoPCR** (In silico PCR used to build the non-elasmobranch part of the reference database from EMBL-EBI release R117)

## Examples

```
ecotag -m 0.0 --ref-database elasmobranch_coi_ref.fasta --sequence-file motu_reps.fasta --results-file ecotag_assignments.fasta
```

## Evaluation signals

- The assignment table exists and contains exactly one row per MOTU with taxonomy fields populated
- All elasmobranch taxa contributing to the 21 shark species reported in the study are present in the ecotag assignments
- MOTUs with 100% sequence identity matches to more than one BOLD species are flagged/handled (e.g. split into Pacific and Caribbean components) rather than silently assigned to one species
- Every representative sequence receives the most specific consistent taxon — no assignment contradicts the reference taxonomy shared by its matches
- Reference database formatting (obitax/ecoPCR-format) completed without errors so ecotag resolves taxonomic parent ranks correctly

## Limitations

- The 127 bp COI fragment lacks resolution to separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to individuals of different species in BOLD, including species from different ocean regions
- Resolving closely related species would require designing alternative primers amplifying a longer fragment — not done in this study
- Trade-off between taxonomic resolution and non-target amplification must be balanced before using this approach to inform conservation decisions

## Evidence

- [discussion] Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75.: "Taxonomic assignment of the representative sequences for each Molecular Operational Taxonomic Unit (MOTU) was performed using the ecotag algorithm75."
- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database: "the sequences of some MOTUs are 100% identical to individuals belonging to different species in the BOLD database"
- [discussion] This combined reference database is available from http://github.com/metabarpark/Reference-databases.: "This combined reference database is available from http://github.com/metabarpark/Reference-databases."
- [discussion] In order to resolve the issue with closely related species, it will be essential to design alternative primers that are able to amplify a longer fragment of the gene region: "In order to resolve the issue with closely related species, it will be essential to design alternative primers that are able to amplify a longer fragment of the gene region"
