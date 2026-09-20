---
name: motu-single-read-filtering
description: Use when a sumaclust-delimited, ecotag-annotated MOTU table (per-sample
  read counts) must be cleaned of single-read MOTUs prior to per-location or per-region
  richness estimates for elasmobranch or other metabarcoding taxa.
when_to_use_negative:
- Input is raw reads that have not yet been aligned, length-filtered (120–135 bp),
  dereplicated and clustered into MOTUs — the filter is defined on clustered MOTU
  tables, not FASTQ.
- The analysis goal is to retain rare-taxa signal or estimate detection prevalence
  of very low-abundance taxa; singleton removal deliberately discards such evidence.
- MOTU clustering used a similarity threshold other than 99% or a tool other than
  the OBITools/sumaclust pipeline, since per-location richness figures are only reproducible
  under this exact workflow.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: obigrep
  role: Discards every MOTU supported by only a single read from the ecotag assignment
    table
  repo: http://github.com/metabarpark
- name: sumaclust
  role: Upstream MOTU delimitation at a constant 99% similarity threshold that produces
    the table the filter operates on
  repo: http://github.com/metabarpark/Reference-databases
- name: ecotag
  role: Provides the taxonomic assignment of representative MOTU sequences used to
    subset to elasmobranch MOTUs after filtering
  repo: http://github.com/metabarpark/Reference-databases
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:72f8b23f65a1957054da84fe4f15e530abfa31bf2eea86c8c2cbe7aab07e25b8
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ecotag
  - obigrep
  - sumaclust
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# motu-single-read-filtering

## Summary

Conservative decontamination step that removes every MOTU supported by only a single read from an OBITools ecotag taxonomic-assignment table before computing elasmobranch MOTU richness, guarding against contamination- and sequencing-error false positives in 99% sumaclust clustering results.

## When to use

Use when a sumaclust-delimited, ecotag-annotated MOTU table (per-sample read counts) must be cleaned of single-read MOTUs prior to per-location or per-region richness estimates for elasmobranch or other metabarcoding taxa.

## When NOT to use

- Input is raw reads that have not yet been aligned, length-filtered (120–135 bp), dereplicated and clustered into MOTUs — the filter is defined on clustered MOTU tables, not FASTQ.
- The analysis goal is to retain rare-taxa signal or estimate detection prevalence of very low-abundance taxa; singleton removal deliberately discards such evidence.
- MOTU clustering used a similarity threshold other than 99% or a tool other than the OBITools/sumaclust pipeline, since per-location richness figures are only reproducible under this exact workflow.

## Inputs

- ecotag taxonomic assignment table (MOTUs from sumaclust at 99% similarity) with per-sample read counts
- bespoke elasmobranch COI reference database built from BOLD plus in silico PCR (ecoPCR, EMBL-EBI R117) non-elasmobranch sequences

## Outputs

- filtered MOTU table with all single-read MOTUs removed
- per-location and per-region MOTU richness summary (e.g. motu_richness.csv): 22 total MOTUs, 12 Caribbean, 16 New Caledonia, 9 shared

## How to apply

Start from the ecotag taxonomic-assignment table of MOTUs delimited with sumaclust at a 99% similarity threshold, with per-sample read counts. Apply obigrep from the OBITools suite to discard every MOTU supported by only a single read, the paper's conservative control against contamination and sequencing-error false positives. Subset the retained MOTUs to elasmobranch assignments against the bespoke BOLD/elasmobranch reference database (github.com/metabarpark/Reference-databases). Compute richness per location (Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5) and per region (12 Caribbean, 16 New Caledonia, 9 shared, 22 total) and write the filtered table and summary (e.g. motu_richness.csv). Accept the output only if all counts reproduce the expected figures exactly, with highest richness in the remote/protected localities (Chesterfield and Bahamas/Bimini).

## Related tools

- **obigrep** (Discards every MOTU supported by only a single read from the ecotag assignment table) — http://github.com/metabarpark
- **sumaclust** (Upstream MOTU delimitation at a constant 99% similarity threshold that produces the table the filter operates on) — http://github.com/metabarpark/Reference-databases
- **ecotag** (Provides the taxonomic assignment of representative MOTU sequences used to subset to elasmobranch MOTUs after filtering) — http://github.com/metabarpark/Reference-databases

## Examples

```
obigrep -p 'sum(count)>1' motus_99_ecotag.fasta > motus_nosingleton.fasta
```

## Evaluation signals

- No MOTU with total read support of one remains in the filtered table.
- Region-level richness after filtering reproduces the reported figures exactly: 22 unique elasmobranch MOTUs overall, 12 in the Caribbean, 16 in New Caledonia, 9 shared.
- Per-location richness matches Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5.
- Highest richness occurs in the remote/protected localities (Chesterfield atolls and Bahamas/Bimini), consistent with the paper's diversity-impact pattern.
- Filtered table plus richness summary are written (e.g. motu_richness.csv) and values are diffed against the expected counts before acceptance.

## Limitations

- Contamination or sequencing error cannot be completely excluded as the cause of single-read MOTUs, so filtering trades sensitivity for specificity against potential false positives.
- The 127 bp COI fragment cannot always separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple BOLD species, capping taxonomic resolution of the richness counts.
- Species with primer mismatches (e.g. Ginglymostoma cirratum observed visually but with no reads recovered) remain undetected regardless of filtering, so richness is a lower bound.

## Evidence

- [discussion] we have opted for a more conservative approach and have removed all single read MOTUs from our samples.: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] it is not possible to completely exclude contamination (or sequencing error) as the potential cause of MOTUs containing only a single read: "it is not possible to completely exclude contamination (or sequencing error) as the potential cause of MOTUs containing only a single read"
- [readme] Reference databases in fasta format for Eukaryotic metabarcoding: "Reference databases in fasta format for Eukaryotic metabarcoding"
