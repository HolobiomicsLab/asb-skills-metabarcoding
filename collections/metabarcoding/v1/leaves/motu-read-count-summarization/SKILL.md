---
name: motu-read-count-summarization
description: Use when a filtered DADA2 MOTU table (detections in >=2 PCR replicates)
  with taxonomic assignments must be reduced to total arthropod reads per marker and
  richness counts at species, family, and order ranks.
when_to_use_negative:
- The MOTU table has not been filtered to at least two PCR replicates — richness and
  read totals will not reproduce the reported values
- Taxonomy assignments have not been made under the 99-100% identity rule, so MOTUs
  cannot be aggregated to species rank
- The question concerns community clustering by plant species rather than read totals
  and richness summarization
edam_topics:
- http://edamontology.org/topic_0632
tools:
- name: DADA2
  role: Produces the MOTU count table and provides taxonomy assignment that this skill
    summarizes
- name: tobiasgf/Bioinformatic-tools
  role: Repository of bioinformatic tooling supporting the metabarcoding workflow
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:8a0240135e30215ed3ce0f6b611a67ab049c071deb84b34a2598a0ebb031d700
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - tobiasgf/Bioinformatic-tools
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0632
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# motu-read-count-summarization

## Summary

Summarizes a DADA2-inferred, replicate-filtered MOTU table into per-marker arthropod read totals and taxonomic richness counts (species/family/order), plus per-species read-share fractions to quantify community dominance. This reproduces the headline numbers of a flower eDNA metabarcoding study.

## When to use

Use when a filtered DADA2 MOTU table (detections in >=2 PCR replicates) with taxonomic assignments must be reduced to total arthropod reads per marker and richness counts at species, family, and order ranks.

## When NOT to use

- The MOTU table has not been filtered to at least two PCR replicates — richness and read totals will not reproduce the reported values
- Taxonomy assignments have not been made under the 99-100% identity rule, so MOTUs cannot be aggregated to species rank
- The question concerns community clustering by plant species rather than read totals and richness summarization

## Inputs

- DADA2-inferred MOTU count table filtered to MOTUs detected in at least 2 PCR replicates
- MOTU taxonomy assignments from arthropod reference databases (species-level requires 99-100% sequence identity)
- Marker-set metadata distinguishing the mitochondrial 16S rRNA and COI amplicons

## Outputs

- Total arthropod read counts per marker (e.g. 23,517,933 COI reads; 658,159 16S reads)
- Taxonomic richness counts: number of arthropod species, families, and orders (e.g. 135 species / 67 families / 14 orders)
- Per-species read-share fractions quantifying dominance (e.g. Thrips major 30% of COI reads; Meligethes planiusculus 51% of 16S reads)

## How to apply

Load the DADA2-inferred MOTU table for the two marker sets targeting mitochondrial 16S rRNA and COI, retaining MOTUs only when detected in at least 2 PCR replicates. Assign taxonomy by matching MOTU sequences against arthropod reference databases, applying the rule that species-level assignment requires a 99-100% sequence-identity match. Filter to arthropod assignments and tally total reads per marker, expecting on the order of 23,517,933 COI reads and 658,159 16S reads for the published flower dataset. Aggregate assigned MOTUs into richness counts at three ranks — reported as at least 135 arthropod species in 67 families and 14 orders. Finally compute per-species read-share fractions to quantify dominance (Thrips major at 30% of COI reads, Meligethes planiusculus at 51% of 16S reads), flagging any marker whose total deviates from reported figures.

## Related tools

- **DADA2** (Produces the MOTU count table and provides taxonomy assignment that this skill summarizes)
- **tobiasgf/Bioinformatic-tools** (Repository of bioinformatic tooling supporting the metabarcoding workflow) — tobiasgf/Bioinformatic-tools

## Evaluation signals

- Per-marker arthropod read totals match the reported 23,517,933 COI reads and 658,159 16S reads
- Richness counts reproduce at least 135 arthropod species, 67 families, and 14 orders
- Dominance shares reproduce Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads
- Every included MOTU is confirmed as detected in at least 2 PCR replicates before counting
- Flag any marker whose arthropod read total deviates from the reported figure

## Limitations

- The study reports 'at least' 135 species — richness is a lower bound since the experiment was not exhaustive and higher richness could be obtained with this eDNA approach
- Totals depend on the study's taxonomic assignment rule (99-100% identity for species); different thresholds change species/family/order counts

## Evidence

- [abstract] eDNA from at least 135 arthropod species in 67 families and 14 orders was recovered: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] two primer sets targeting mitochondrial 16S rRNA and COI: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] final replicate-filtered yield of 135 species in 67 families and 14 orders: [evidence span withheld — anchor status: not_found]
