---
name: metabarcoding-contamination-control
description: Use when a MOTU table built with sumaclust (99% similarity) contains
  single-read MOTUs and the goal is a conservative contamination/false-positive control
  before computing elasmobranch (or other taxon) richness per sampling location or
  region.
when_to_use_negative:
- The input is raw FASTQ reads — singleton removal operates on dereplicated, clustered
  MOTU tables, not reads; quality/length filtering happens earlier with illuminapairedend,
  obigrep length filtering (120–135 bp), and vsearch chimera removal
- The analysis aims to detect genuinely rare, low-abundance taxa where single-read
  evidence must be retained — the paper explicitly accepts losing such signals as
  the price of a conservative control
- MOTUs were not clustered with sumaclust and assigned with ecotag — the singleton
  definition (MOTU supported by exactly one read) depends on that per-MOTU read-count
  structure
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3911
tools:
- name: obigrep
  role: Removes every MOTU supported by only a single read (singleton removal) from
    the assignment table
  repo: https://github.com/metabarpark/Reference-databases
- name: ecotag
  role: Provides the taxonomic assignment table (MOTU-to-reference assignments) on
    which singleton filtering and elasmobranch subsetting are performed
- name: sumaclust
  role: Delimits MOTUs at a constant 99% similarity threshold, defining the units
    whose read counts are screened for singletons
- name: ecoPCR
  role: Used for in silico PCR against EMBL-EBI release R117 to build the non-elasmobranch
    part of the reference database used for assignment
  repo: https://github.com/metabarpark/Reference-databases
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:1bf5c4117bfa639afc4c63c7d3ba7de4a40eeed658daa1cd2baa3ac590e1bd94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/metabarpark/Reference-databases
  tools:
  - ecoPCR
  - ecotag
  - obigrep
  - sumaclust
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3911
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# metabarcoding-contamination-control

## Summary

A conservative metabarcoding quality-control skill that removes all single-read (singleton) MOTUs from an ecotag taxonomic assignment table to control contamination and sequencing-error false positives, then recounts per-location and per-region MOTU richness on the filtered set.

## When to use

Use when a MOTU table built with sumaclust (99% similarity) contains single-read MOTUs and the goal is a conservative contamination/false-positive control before computing elasmobranch (or other taxon) richness per sampling location or region.

## When NOT to use

- The input is raw FASTQ reads — singleton removal operates on dereplicated, clustered MOTU tables, not reads; quality/length filtering happens earlier with illuminapairedend, obigrep length filtering (120–135 bp), and vsearch chimera removal
- The analysis aims to detect genuinely rare, low-abundance taxa where single-read evidence must be retained — the paper explicitly accepts losing such signals as the price of a conservative control
- MOTUs were not clustered with sumaclust and assigned with ecotag — the singleton definition (MOTU supported by exactly one read) depends on that per-MOTU read-count structure

## Inputs

- ecotag taxonomic assignment table with per-sample read counts (MOTUs delimited by sumaclust at 99% similarity)
- bespoke elasmobranch reference database built from BOLD COI sequences plus in silico PCR non-elasmobranch sequences (ecoPCR vs EMBL-EBI release R117)
- sample metadata mapping MOTU reads to sampling locations (Bahamas, Jamaica, Belize, Chesterfield, New Caledonia North, Noumea)

## Outputs

- filtered MOTU table with all single-read MOTUs removed
- MOTU richness summary per location and per region (motu_richness.csv)

## How to apply

Start from the ecotag taxonomic assignment table for MOTUs delimited with sumaclust at a 99% similarity threshold, keeping per-sample read counts. Apply obigrep from the OBITools suite to discard every MOTU supported by only a single read, mirroring the paper's conservative control against contamination and sequencing-error false positives. Subset the retained MOTUs to elasmobranch assignments against the bespoke BOLD/elasmobranch reference database (github.com/metabarpark/Reference-databases). Compute MOTU richness per location (Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5) and per region (12 Caribbean, 16 New Caledonia, 9 shared; 22 MOTUs overall). The rationale is that a MOTU backed by a single read cannot be distinguished from a contamination or sequencing-error artifact, so it is removed even at the risk of losing true low-abundance detections.

## Related tools

- **obigrep** (Removes every MOTU supported by only a single read (singleton removal) from the assignment table) — https://github.com/metabarpark/Reference-databases
- **ecotag** (Provides the taxonomic assignment table (MOTU-to-reference assignments) on which singleton filtering and elasmobranch subsetting are performed)
- **sumaclust** (Delimits MOTUs at a constant 99% similarity threshold, defining the units whose read counts are screened for singletons)
- **ecoPCR** (Used for in silico PCR against EMBL-EBI release R117 to build the non-elasmobranch part of the reference database used for assignment) — https://github.com/metabarpark/Reference-databases

## Examples

```
obigrep -p 'count>1' motus_ecotag.fastx > motus_singletons_removed.fastx
```

## Evaluation signals

- Singleton removal yields exactly 22 elasmobranch MOTUs in total after filtering
- Region-level distribution matches 12 MOTUs in the Caribbean, 16 in New Caledonia, and 9 shared between the two locations
- Per-location richness values match Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5
- Highest richness occurs in the remote/protected localities (Chesterfield atolls and Bimini/Bahamas), consistent with the reported diversity pattern
- No MOTU in the filtered table has a total read count of 1

## Limitations

- Single-read MOTUs may be genuine detections of rare taxa; removing them trades sensitivity for specificity
- Contamination (or sequencing error) cannot be completely excluded as the cause of single-read MOTUs, so the filter is a conservative control rather than proof of artifact removal
- The 127 bp COI fragment cannot separate closely related species (e.g. Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple BOLD species, affecting taxonomic interpretation of the richness counts
- Some elasmobranch species present at sampling could not be detected due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered)

## Evidence

- [discussion] we have opted for a more conservative approach and have removed all single read MOTUs from our samples.: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] it is not possible to completely exclude contamination (or sequencing error) as the potential cause of MOTUs containing only a single read: "it is not possible to completely exclude contamination (or sequencing error) as the potential cause of MOTUs containing only a single read"
- [methods] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [readme] If you use the reference database for COI elasmobranch sequences using Fields et al. (2015) primers, please cite:: "If you use the reference database for COI elasmobranch sequences using Fields et al. (2015) primers, please cite:"
