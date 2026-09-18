---
name: obitools-command-line-workflow-execution
description: Use when processing eDNA metabarcoding FASTQ data (e.g. elasmobranch
  COI amplicons) through OBITools and need to reproduce per-MOTU richness after removing
  single-read MOTUs.
when_to_use_negative:
- Input is already a denoised, chimera-checked MOTU table — skip the OBITools read-processing
  steps and go straight to singleton filtering and richness counting
- Data are 12S or 16S shark metabarcoding data whose expected fragment length is not
  the 120–135 bp COI window this pipeline's obigrep filter enforces
edam_operation: http://edamontology.org/operation_0238
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: OBITools
  role: Metabarcoding software suite driving the whole read-processing and MOTU-delimitation
    workflow
  repo: http://github.com/metabarpark
- name: obigrep
  role: 'Applied twice: 120–135 bp length filter on aligned reads, and removal of
    all single-read MOTUs'
- name: sumaclust
  role: MOTU delimitation with a constant similarity threshold of 99%
- name: ecotag
  role: Taxonomic assignment of MOTU representative sequences against the bespoke
    BOLD/elasmobranch reference database
- name: vsearch
  role: Chimera removal via the uchime-denovo algorithm
- name: ecoPCR
  role: In silico PCR against EMBL-EBI release R117 to build the reference database
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:8c5486b5e0452f5ebd38638d572915eb4d14b7b926946a869281bf96036456d4
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - ecoPCR
  - ecotag
  - obigrep
  - sumaclust
  - vsearch
  edam_operation: http://edamontology.org/operation_0238
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# obitools-command-line-workflow-execution

## Summary

Execute the OBITools metabarcoding command-line pipeline used by Bakke et al. (2017) to go from raw Illumina MiSeq reads to ecotag-assigned MOTUs, then apply the paper's conservative singleton-removal filter (obigrep) before counting elasmobranch MOTU richness per location and region.

## When to use

Use when processing eDNA metabarcoding FASTQ data (e.g. elasmobranch COI amplicons) through OBITools and need to reproduce per-MOTU richness after removing single-read MOTUs.

## When NOT to use

- Input is already a denoised, chimera-checked MOTU table — skip the OBITools read-processing steps and go straight to singleton filtering and richness counting
- Data are 12S or 16S shark metabarcoding data whose expected fragment length is not the 120–135 bp COI window this pipeline's obigrep filter enforces

## Inputs

- Raw paired-end Illumina MiSeq FASTQ reads from eDNA seawater samples
- Sample metadata / sequencing key for ngsfilter demultiplexing (tagged 8-base primer protocol)
- Bespoke elasmobranch reference database (BOLD COI sequences plus non-elasmobranch sequences from in silico PCR on EMBL-EBI R117), FASTA format
- ecotag taxonomic assignment table with per-sample read counts per MOTU

## Outputs

- Length-filtered, dereplicated, chimera-free aligned read table
- MOTU table delimited with sumaclust at 99% similarity, with ecotag taxonomic assignments and per-sample read counts
- Singleton-filtered MOTU table (single-read MOTUs removed)
- Per-location and per-region MOTU richness summary (e.g. motu_richness.csv)

## How to apply

Run the OBITools suite step by step: align paired-end reads with illuminapairedend keeping alignments with quality score >40, demultiplex with ngsfilter, length-filter with obigrep to 120–135 bp target fragments, remove reads with ambiguous bases, dereplicate with obiuniq, remove chimeras with uchime-denovo in vsearch, and cluster MOTUs with sumaclust at a constant 99% similarity threshold. Assign taxonomy with ecotag against the bespoke elasmobranch reference database built from BOLD COI sequences plus non-elasmobranch sequences from in silico PCR (ecoPCR against EMBL-EBI release R117). Then apply the paper's conservative control: discard every MOTU supported by only a single read using obigrep, to exclude contamination and sequencing-error false positives. Subset retained MOTUs to elasmobranch assignments and compute MOTU richness per locality (e.g. Bahamas 11, Chesterfield 11, New Caledonia North 14) and per region, writing the filtered MOTU table and richness summary (e.g. motu_richness.csv). Accept the result only if region-level totals match the reported 12 Caribbean / 16 New Caledonia / 9 shared / 22 total MOTUs.

## Related tools

- **OBITools** (Metabarcoding software suite driving the whole read-processing and MOTU-delimitation workflow) — http://github.com/metabarpark
- **obigrep** (Applied twice: 120–135 bp length filter on aligned reads, and removal of all single-read MOTUs)
- **sumaclust** (MOTU delimitation with a constant similarity threshold of 99%)
- **ecotag** (Taxonomic assignment of MOTU representative sequences against the bespoke BOLD/elasmobranch reference database)
- **vsearch** (Chimera removal via the uchime-denovo algorithm)
- **ecoPCR** (In silico PCR against EMBL-EBI release R117 to build the reference database)

## Examples

```
obigrep -p 'count>1' motus_ecotag.fasta > motus_singletons_removed.fasta
```

## Evaluation signals

- Region-level richness totals after singleton removal reproduce the reported figures exactly: 12 MOTUs in the Caribbean, 16 in New Caledonia, 9 shared, 22 unique elasmobranch MOTUs overall
- Per-location richness matches reported values: Bahamas 11, Jamaica 2, Belize 1, Chesterfield 11, New Caledonia North 14, Noumea 5
- Highest MOTU richness occurs in the remote/protected localities (Chesterfield atolls and Bimini/Bahamas), consistent with the paper's diversity-impact finding
- The length-filtered read distribution falls entirely within 120–135 bp; no reads with ambiguous bases remain after filtering
- The final MOTU table contains zero MOTUs with a total read count of 1

## Limitations

- The 127 bp COI fragment cannot separate closely related species (Carcharhinus, Rhizoprionodon, Negaprion); some MOTUs are 100% identical to multiple BOLD species, including species from different ocean regions
- Some elasmobranch species present at sampling cannot be detected due to primer mismatches (e.g. Ginglymostoma cirratum observed visually but no reads recovered)
- Contamination or sequencing error cannot be fully excluded as the cause of single-read MOTUs, which motivates but also biases the singleton-removal step
- COI's high sequence variability and primer bias mean read abundance should not be interpreted quantitatively as biomass without dedicated experiments

## Evidence

- [methods] The bioinformatic analysis was based on the OBITools metabarcoding software suite75.: "The bioinformatic analysis was based on the OBITools metabarcoding software suite75."
- [methods] MOTUs delimited with sumaclust at 99%: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [results] Singleton removal as conservative control: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
- [methods] Length filter parameters: "a length filter (obigrep) was applied to the aligned reads (120–135 bp) in order to select only the fragments with the correct target size."
- [readme] Reference database provenance: "If you use the reference database for COI elasmobranch sequences using Fields et al. (2015) primers, please cite: Bakker et al. (2017)"
