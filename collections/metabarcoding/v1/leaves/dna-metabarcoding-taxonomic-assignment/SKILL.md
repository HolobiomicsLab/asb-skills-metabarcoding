---
name: dna-metabarcoding-taxonomic-assignment
description: Use when ASV/MOTU tables from amplicon metabarcoding (e.g. DADA2 output
  for 16S rRNA or COI markers) need taxonomic assignment, arthropod-only filtering,
  and taxonomic richness or dominance summaries.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been denoised — run DADA2 MOTU inference
  first; this skill starts from the inferred MOTU table
- The analysis targets non-arthropod taxa or requires presence/absence occupancy modeling
  rather than taxonomic assignment and read-based richness/dominance summaries
edam_operation: http://edamontology.org/operation_3460
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: DADA2
  role: Amplicon denoising and MOTU inference; its output table (replicate-filtered)
    is the direct input to taxonomy assignment
- name: tobiasgf/Bioinformatic-tools
  role: Repository of bioinformatic scripts used for sequence processing and taxonomic
    assignment in the flower eDNA pipeline
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:b7780b328468623cad3129dbe209ec73a976885fa62b879055950b627d2ada8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - tobiasgf/Bioinformatic-tools
  edam_operation: http://edamontology.org/operation_3460
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# dna-metabarcoding-taxonomic-assignment

## Summary

Assigns taxonomy to DADA2-inferred eDNA MOTUs by matching amplicon sequences against curated arthropod reference databases for two markers (mitochondrial 16S rRNA and COI), then aggregates assignments into species/family/order richness and per-taxon read-share metrics. It underpins the flower-eDNA result of at least 135 arthropod species in 67 families and 14 orders.

## When to use

Use when ASV/MOTU tables from amplicon metabarcoding (e.g. DADA2 output for 16S rRNA or COI markers) need taxonomic assignment, arthropod-only filtering, and taxonomic richness or dominance summaries.

## When NOT to use

- Input is raw FASTQ reads that have not yet been denoised — run DADA2 MOTU inference first; this skill starts from the inferred MOTU table
- The analysis targets non-arthropod taxa or requires presence/absence occupancy modeling rather than taxonomic assignment and read-based richness/dominance summaries

## Inputs

- DADA2-inferred MOTU/ASV count table filtered to MOTUs detected in at least 2 PCR replicates
- MOTU representative sequences (FASTA) for 16S rRNA and COI markers
- Arthropod reference sequence databases for 16S rRNA and COI

## Outputs

- Per-MOTU taxonomic assignment table (species/family/order with sequence-identity support)
- Total arthropod read counts per marker (e.g. 23,517,933 COI reads; 658,159 16S reads)
- Taxonomic richness summary: number of species, families, and orders (135 species / 67 families / 14 orders)
- Per-species read-share fractions quantifying dominance (e.g. Thrips major 30% of COI reads; Meligethes planiusculus 51% of 16S reads)

## How to apply

Load the DADA2-inferred MOTU table (MOTUs retained only when detected in at least 2 PCR replicates) for the 16S rRNA and COI marker sets. Match each MOTU sequence against arthropod reference databases, applying the paper's rule that a species-level assignment requires a 99-100% sequence-identity match. Filter assignments to arthropods and tally total arthropod reads per marker, expecting on the order of 23,517,933 COI reads and 658,159 16S reads for this dataset. Aggregate assigned MOTUs into taxonomic richness counts (species, family, order) and compute per-species read-share fractions to quantify dominance. Decision point: assign to species only when the 99-100% identity threshold is met; otherwise retain coarser-level assignments (family/order) and report richness as a minimum ('at least 135 species').

## Related tools

- **DADA2** (Amplicon denoising and MOTU inference; its output table (replicate-filtered) is the direct input to taxonomy assignment)
- **tobiasgf/Bioinformatic-tools** (Repository of bioinformatic scripts used for sequence processing and taxonomic assignment in the flower eDNA pipeline) — tobiasgf/Bioinformatic-tools

## Evaluation signals

- Total arthropod read counts per marker match the reported figures (23,517,933 COI reads; 658,159 16S reads); flag any marker whose arthropod read total deviates
- Taxonomic richness tallies reproduce 135 arthropod species, 67 families, and 14 orders
- Dominance read-shares reproduce Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads
- Species-level assignments are supported by 99-100% sequence-identity matches; any lower-identity assignment must not be counted at species level
- Every MOTU counted in the richness summary appears in at least 2 PCR replicates in the input MOTU table

## Limitations

- Richness is a lower bound — the experiment was not exhaustive and higher arthropod richness could be obtained with this eDNA approach
- Species-level resolution depends on reference database coverage; without a 99-100% identity match, MOTUs can only be reported at coarser ranks
- Read-share dominance metrics reflect amplification and sequencing biases between the two markers (16S vs COI), not absolute abundance

## Evidence

- [abstract] eDNA from at least 135 arthropod species in 67 families and 14 orders: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups"
- [abstract] Two primer sets targeting 16S and COI: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [results] Final replicate-filtered richness results: [evidence span withheld — anchor status: not_found]
