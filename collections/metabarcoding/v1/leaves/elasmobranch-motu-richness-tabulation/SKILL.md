---
name: elasmobranch-motu-richness-tabulation
description: Use when you have a MOTU table with per-sample read counts and taxonomic
  assignments (ecotag output from sumaclust-delimited MOTUs) and need post-singleton-removal
  richness counts per location or region for a taxon subset such as elasmobranchs.
when_to_use_negative:
- Input is raw FASTQ reads — singleton removal operates on a dereplicated, clustered,
  taxonomically assigned MOTU table, not on reads.
- The analysis goal is species-level identification of closely related taxa (e.g.
  Carcharhinus, Rhizoprionodon, Negaprion) — the 127 bp COI fragment leaves some MOTUs
  100% identical to multiple BOLD species, so richness counts cannot be equated with
  species counts.
- Quantitative biomass/abundance estimation is desired — read abundance is not a validated
  proxy for elasmobranch community biomass.
- Input is already a richness table or the MOTUs have not been clustered and assigned
  (e.g. chimeras not yet removed via vsearch uchime-denovo).
edam_operation: http://edamontology.org/operation_0325
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: obigrep
  role: filters out all single-read MOTUs from the MOTU table (singleton removal)
- name: ecotag
  role: provides the taxonomic assignment used to subset retained MOTUs to elasmobranchs
- name: sumaclust
  role: delimits the MOTUs at the 99% similarity threshold underlying the table being
    filtered
- name: OBITools
  role: suite providing obigrep and ecotag for the filtering and assignment steps
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:c102e586b1fde215f4a647d672a74f1be9faf6e15bb255cffff91c2a873eb1db
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecotag
  - obigrep
  - sumaclust
  edam_operation: http://edamontology.org/operation_0325
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# elasmobranch-motu-richness-tabulation

## Summary

Tabulates elasmobranch MOTU richness per sampling location and region from an ecotag taxonomic assignment table after removing single-read (singleton) MOTUs, as a conservative contamination/sequencing-error control. Produces location-level counts (e.g. Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5) and region-level totals (12 Caribbean, 16 New Caledonia, 9 shared, 22 overall).

## When to use

Use when you have a MOTU table with per-sample read counts and taxonomic assignments (ecotag output from sumaclust-delimited MOTUs) and need post-singleton-removal richness counts per location or region for a taxon subset such as elasmobranchs.

## When NOT to use

- Input is raw FASTQ reads — singleton removal operates on a dereplicated, clustered, taxonomically assigned MOTU table, not on reads.
- The analysis goal is species-level identification of closely related taxa (e.g. Carcharhinus, Rhizoprionodon, Negaprion) — the 127 bp COI fragment leaves some MOTUs 100% identical to multiple BOLD species, so richness counts cannot be equated with species counts.
- Quantitative biomass/abundance estimation is desired — read abundance is not a validated proxy for elasmobranch community biomass.
- Input is already a richness table or the MOTUs have not been clustered and assigned (e.g. chimeras not yet removed via vsearch uchime-denovo).

## Inputs

- ecotag taxonomic assignment table with per-sample read counts (MOTUs delimited by sumaclust at 99% similarity threshold)
- bespoke elasmobranch COI reference database from BOLD plus non-elasmobranch in silico PCR sequences (github.com/metabarpark/Reference-databases)

## Outputs

- filtered MOTU table after single-read MOTU removal
- MOTU richness summary per location and per region (e.g. motu_richness.csv)

## How to apply

Load the ecotag taxonomic assignment table where MOTUs were delimited with sumaclust at a constant 99% similarity threshold, with per-sample read counts attached. Use obigrep from the OBITools suite to discard every MOTU supported by only a single read — the paper's conservative control against contamination and sequencing-error false positives. Subset the retained MOTUs to those assigned against the bespoke elasmobranch reference database built from BOLD COI sequences plus in silico PCR non-elasmobranch sequences (ecoPCR against EMBL-EBI release R117). Compute MOTU richness per locality and aggregate per region, distinguishing shared versus unique MOTUs between the Caribbean and New Caledonia sampling regions. Validate by checking region-level totals reproduce the reported figures (22 total: 12 Caribbean, 16 New Caledonia, 9 shared) and that highest richness occurs in remote/protected localities such as Chesterfield and the Bahamas/Bimini.

## Related tools

- **obigrep** (filters out all single-read MOTUs from the MOTU table (singleton removal))
- **ecotag** (provides the taxonomic assignment used to subset retained MOTUs to elasmobranchs)
- **sumaclust** (delimits the MOTUs at the 99% similarity threshold underlying the table being filtered)
- **OBITools** (suite providing obigrep and ecotag for the filtering and assignment steps)

## Examples

```
obigrep -p 'count>1' motus_ecotag.fasta > motus_nosingletons.fasta
```

## Evaluation signals

- After obigrep singleton removal, elasmobranch MOTU richness totals must reproduce the reported figures: 22 MOTUs overall, 12 Caribbean, 16 New Caledonia, 9 shared.
- Per-locality richness must match: Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5.
- Highest richness should appear in remote/protected localities (Chesterfield atolls and Bimini/Bahamas), consistent with the paper's diversity–anthropogenic-impact pattern.
- Every retained MOTU must have read count > 1 in the filtered table; no MOTU supported by a single read may remain.
- The filtered table should only contain MOTUs with elasmobranch assignments against the bespoke BOLD/elasmobranch reference database.

## Limitations

- Contamination or sequencing error cannot be completely excluded as the cause of single-read MOTUs, so singleton removal is a conservative but imperfect control against false positives.
- Taxonomic resolution of the 127 bp COI fragment is insufficient to separate closely related species; some MOTUs are 100% identical to multiple species in BOLD, including species from different ocean regions, so MOTU counts are not species counts.
- Certain elasmobranch species present at sampling could not be detected with the selected primer set (e.g. Ginglymostoma cirratum observed visually but no reads recovered), undercounting true richness.
- Read abundance per MOTU cannot be interpreted quantitatively without accounting for primer bias and differing shedding rates.

## Evidence

- [discussion] singleton removal is the conservative control: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] MOTU delimitation basis of the table: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] expected richness pattern for validation: "Remote localities such as the Chesterfield atolls and protected areas such as Bimini, Bahamas show both the highest species richness and read abundance among our samples"
- [readme] reference database source: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
