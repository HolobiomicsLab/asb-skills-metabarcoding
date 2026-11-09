---
name: ecotaxonomic-assignment-table-processing
description: Use when you have an ecotag-style per-sample MOTU read-count table and
  need to apply a conservative singleton-removal filter before reporting taxon richness
  per sampling location or region.
when_to_use_negative:
- Input is raw sequencing reads that have not yet been aligned, length-filtered (120-135
  bp), dereplicated, and chimera-removed — this skill operates on the post-clustering
  ecotag table, not FASTQ.
- The goal is a permissive richness estimate including rare reads — removing all single-read
  MOTUs is by design a conservative control and will discard low-support detections.
- MOTUs were clustered at a similarity threshold other than 99% with sumaclust — richness
  counts will not be comparable to the paper's reported figures.
edam_operation: http://edamontology.org/operation_3695
tools:
- name: obigrep
  role: Discards MOTUs supported by only a single read (singleton-removal filter on
    the assignment table)
- name: ecotag
  role: Taxonomic assignment of MOTU representative sequences against the bespoke
    elasmobranch reference database, producing the input table
- name: sumaclust
  role: Delimits MOTUs at a constant 99% similarity threshold upstream of this filtering
    step
- name: OBITools
  role: Metabarcoding software suite providing obigrep and ecotag used in this workflow
- name: Reference-databases
  role: Provides the COI elasmobranch reference database (Fields et al. primers) used
    for taxonomic assignment
  repo: github.com/metabarpark/Reference-databases
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:1c4a68bc68e40dac25757fbf2c2f45915e8c3d4eebdf84c75ab7562e367ad0da
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - Reference-databases
  - ecotag
  - obigrep
  - sumaclust
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# ecotaxonomic-assignment-table-processing

## Summary

Filters an ecotag taxonomic assignment table produced from sumaclust MOTU clustering (99% similarity threshold) by removing all single-read (singleton) MOTUs, then computes per-location and per-region elasmobranch MOTU richness. This is the paper's conservative control against contamination and sequencing-error false positives.

## When to use

Use when you have an ecotag-style per-sample MOTU read-count table and need to apply a conservative singleton-removal filter before reporting taxon richness per sampling location or region.

## When NOT to use

- Input is raw sequencing reads that have not yet been aligned, length-filtered (120-135 bp), dereplicated, and chimera-removed — this skill operates on the post-clustering ecotag table, not FASTQ.
- The goal is a permissive richness estimate including rare reads — removing all single-read MOTUs is by design a conservative control and will discard low-support detections.
- MOTUs were clustered at a similarity threshold other than 99% with sumaclust — richness counts will not be comparable to the paper's reported figures.

## Inputs

- ecotag taxonomic assignment table with per-sample read counts for MOTUs delimited at 99% similarity
- bespoke elasmobranch COI reference database (BOLD sequences + ecoPCR non-elasmobranch sequences, EMBL-EBI R117)
- sample-to-location metadata mapping samples to localities and regions (Caribbean vs New Caledonia)

## Outputs

- filtered MOTU table (elasmobranch MOTUs with >1 read, per-sample counts)
- motu_richness.csv with per-location MOTU richness and per-region/shared counts

## How to apply

Load the ecotag taxonomic assignment table, where MOTUs were delimited with sumaclust at a constant 99% similarity threshold and representative sequences assigned against a bespoke elasmobranch reference database (BOLD COI sequences plus in silico PCR non-targets from EMBL-EBI release R117 via ecoPCR). Use obigrep from the OBITools suite with a count>1 predicate to discard every MOTU supported by only a single read, because contamination or sequencing error cannot be excluded for single-read MOTUs. Subset the retained MOTUs to elasmobranch assignments and compute richness per locality (Bahamas, Jamaica, Belize, Chesterfield, New Caledonia North, Noumea) and per region, counting MOTUs shared between the Caribbean and New Caledonia. Write the filtered MOTU table and richness summary to a CSV (motu_richness.csv). The decision point is the singleton threshold itself: the paper deliberately chose a conservative single-read cutoff rather than a higher abundance threshold.

## Related tools

- **obigrep** (Discards MOTUs supported by only a single read (singleton-removal filter on the assignment table))
- **ecotag** (Taxonomic assignment of MOTU representative sequences against the bespoke elasmobranch reference database, producing the input table)
- **sumaclust** (Delimits MOTUs at a constant 99% similarity threshold upstream of this filtering step)
- **OBITools** (Metabarcoding software suite providing obigrep and ecotag used in this workflow)
- **Reference-databases** (Provides the COI elasmobranch reference database (Fields et al. primers) used for taxonomic assignment) — github.com/metabarpark/Reference-databases

## Examples

```
obigrep -p 'count>1' ecotag_assignment.ecotag.fasta > motus_singletons_removed.fasta
```

## Evaluation signals

- Region-level totals reproduce the reported figures exactly: 12 MOTUs in the Caribbean, 16 in New Caledonia, 9 shared, 22 unique MOTUs overall after singleton removal.
- Per-locality richness matches the reported values: Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5.
- Highest richness occurs in the remote/protected localities (Chesterfield and Bahamas), consistent with the paper's finding that richness follows anthropogenic impact.
- Zero MOTUs with a total read count of 1 remain in the filtered table (verify the obigrep count>1 predicate removed every singleton).
- Output CSV contains both the filtered per-sample MOTU table and a richness summary keyed to location and region.

## Limitations

- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple species in BOLD, including species from different ocean regions.
- Some elasmobranch species present at sampling could not be detected due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered).
- Contamination or sequencing error cannot be completely excluded as the cause of single-read MOTUs, which is why they are removed; this also risks discarding true rare detections.
- Primer efficiency may vary among species templates (primer bias), limiting quantitative interpretation of read counts.

## Evidence

- [discussion] we have opted for a more conservative approach and have removed all single read MOTUs from our samples.: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance: "Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance among our samples"
- [readme] This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences.: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
