---
name: fungal-functional-group-classification
description: Use when you have clustered eukaryotic (ITS) OTU sequences from metabarcoding
  data and need to know which OTUs correspond to known species and how those identified
  OTUs split across Fungi/Plantae/Animalia and fungal functional groups.
when_to_use_negative:
- Input is prokaryotic 16S rRNA reads — these should follow the SILVAngs pipeline
  path, not the ITS classification workflow
- OTUs cannot be assigned to known species (no/weak hits where (% sequence identity
  + % alignment coverage)/2 does not exceed 93), so functional-group labels cannot
  be defensibly assigned
- A pre-computed kingdom/functional-group feature table already exists — regenerating
  percentages from raw reads is redundant
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_0621
tools:
- name: cd-hit-est v3.1.2
  role: Clusters ITS reads into OTUs (identity criteria 1.00 and 0.98) prior to species/functional-group
    assignment
- name: Classify Sequences (Warcup V2)
  role: Assigns taxonomy to ITS consensus sequences using the Warcup V2 training data
    set with medium sensitivity
- name: BLASTn
  role: Local BLASTn search against the non-redundant SILVA SSU ref dataset (release
    no. 132) for taxonomic identification supporting species-level labels
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-020-80236-x
    title: Metabarcoding profiling of microbial diversity associated with trout fish
      farming
  source_content_hash: sha256:316817b77993d50c74817d47ff5cf6016eda37ff3c4986ea16d244161047ef21
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BLASTn
  - Classify Sequences (Warcup V2)
  - cd-hit-est v3.1.2
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_0621
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# fungal-functional-group-classification

## Summary

Assigns eukaryotic ITS2 metabarcoding OTUs to kingdoms and fungal functional groups (plant pathogens, mushroom/yeasts, malodor producers, fish and human pathogens) by first identifying OTUs as known species and then tallying their proportions, as done for the 233 eukaryotic OTUs (118 known species) profiled in the trout-farm water study.

## When to use

Use when you have clustered eukaryotic (ITS) OTU sequences from metabarcoding data and need to know which OTUs correspond to known species and how those identified OTUs split across Fungi/Plantae/Animalia and fungal functional groups.

## When NOT to use

- Input is prokaryotic 16S rRNA reads — these should follow the SILVAngs pipeline path, not the ITS classification workflow
- OTUs cannot be assigned to known species (no/weak hits where (% sequence identity + % alignment coverage)/2 does not exceed 93), so functional-group labels cannot be defensibly assigned
- A pre-computed kingdom/functional-group feature table already exists — regenerating percentages from raw reads is redundant

## Inputs

- Consensus sequences of clustered eukaryotic/ITS2 OTUs (FASTA) from cd-hit-est v3.1.2 clustering at identity 0.98
- Reference taxonomy via the 'Classify Sequences' tool with the Warcup V2 training data set and BLASTn against non-redundant SILVA SSU ref dataset (release no. 132)

## Outputs

- Kingdom-level distribution of identified eukaryotic OTUs (e.g. ~55% Fungi, ~33% Plantae, ~12% Animalia)
- Fungal functional-group profile table of known-species OTUs with counts and percentages (plant pathogens, mushroom/yeasts, volatile producers, fish pathogens, human pathogens), e.g. written to eukaryotic_otu_profile.csv

## How to apply

After ITS2 PCR amplicons are merged (BBmerge), trimmed (Q30, discard <50 bp), clustered into OTUs (cd-hit-est v3.1.2, identity 0.98 for ITS), assemble consensus sequences and assign taxonomy with the 'Classify Sequences' tool using the Warcup V2 training data set, and cross-check via local BLASTn against reference datasets; hits must reach 90% similarity with a minimum overlap of 75 bp. Then restrict to the subset of eukaryotic OTUs identified as known species (in this study 118 of 233) and label each with its functional role — plant pathogen, mushroom/yeast, volatile (malodor) producer, fish pathogen, or human pathogen — plus kingdom (Fungi, Plantae, Animalia). Tally the proportions (here ~55% Fungi, ~33% Plantae, ~12% Animalia; among fungi ~45% plant pathogens, ~28% mushroom/yeasts, ~11% malodor producers, ~8% fish pathogens, ~8% human pathogens) and record them as counts and percentages. The rationale is that species-level identification is required for functional-group labeling, and OTUs without strong reference hits must be left unclassified rather than forced into a group.

## Related tools

- **cd-hit-est v3.1.2** (Clusters ITS reads into OTUs (identity criteria 1.00 and 0.98) prior to species/functional-group assignment)
- **Classify Sequences (Warcup V2)** (Assigns taxonomy to ITS consensus sequences using the Warcup V2 training data set with medium sensitivity)
- **BLASTn** (Local BLASTn search against the non-redundant SILVA SSU ref dataset (release no. 132) for taxonomic identification supporting species-level labels)

## Examples

```
df.to_csv('eukaryotic_otu_profile.csv', index=False)  # columns: category, group, value (233 total OTUs; 118 known species; ~55/33/12% Fungi/Plantae/Animalia)
```

## Evaluation signals

- The count of known-species OTUs (118) is a strict subset of total eukaryotic OTUs (233)
- Kingdom percentages for identified OTUs sum to approximately 100% (~55% Fungi + ~33% Plantae + ~12% Animalia)
- Fungal functional-group percentages fall within the rounded ranges stated: ~45% plant pathogens, ~28% mushroom/yeasts, ~11% volatile producers, ~8% fish pathogens, ~8% human pathogens
- Output CSV is parseable with pandas and numeric fields reproduce 233 total and 118 known-species counts
- Any unclassified OTUs (weak BLAST hits, score ≤ 93) are excluded from the functional-group tally rather than misassigned

## Limitations

- Species-level identification depends on reference coverage; OTUs with no or weak BLAST hits remained unclassified or unassigned
- The ITS classification is constrained by a 90% similarity and 75 bp minimum-overlap threshold, so distant relatives may go unidentified
- Only known-species OTUs can receive functional-group labels; the fungal functional-group profile therefore describes 118 OTUs, not all 233
- The article notes the role of fungi in malodor formation in aquaculture was not sufficiently studied, so functional labels (e.g. malodor producers) carry mechanistic uncertainty

## Evidence

- [abstract] 118 out of 233 of the eukaryotic OTUs were known species. Of these, ~ 45% were plant pathogens, and ~ 28% were mushroom/yeasts: "118 out of 233 of the eukaryotic OTUs were known species. Of these, ~ 45% were plant pathogens, and ~ 28% were mushroom/yeasts"
- [discussion] Generated consensus sequences were subject to the "Classify Sequences" tool using the Warcup V2 training data set: "Generated consensus sequences were subject to the "Classify Sequences" tool using the Warcup V2 training data set"
- [discussion] clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2: "clustered and assigned as OTUs (organism taxonomical units) using cd-hit-est v3.1.2"
- [discussion] Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned: "Samples with no or weak BLAST hits where the function "(% sequence identity + % alignment coverage)/2" did not exceed the value of 93 remained unclassified or unassigned"
