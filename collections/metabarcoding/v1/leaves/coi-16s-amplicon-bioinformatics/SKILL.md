---
name: coi-16s-amplicon-bioinformatics
description: Use when raw 16S/COI amplicon FASTQ data from eDNA or metabarcoding samples
  must be denoised into MOTUs, filtered by PCR-replicate detection, taxonomically
  assigned, and summarized into read counts and taxonomic richness.
when_to_use_negative:
- Input is not paired 16S/COI amplicon data from eDNA samples — e.g., shotgun metagenomes
  or single-marker host-barcode runs where replicate-level MOTU filtering does not
  apply
- The MOTU table is already taxonomically assigned and replicate-filtered; only the
  downstream richness/dominance summarization is needed
edam_operation: http://edamontology.org/operation_0232
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0080
tools:
- name: DADA2
  role: Denoising and MOTU inference from 16S rRNA and COI amplicon reads prior to
    replicate filtering
  repo: tobiasgf/Bioinformatic-tools
- name: tobiasgf/Bioinformatic-tools
  role: Repository hosting the bioinformatic scripts used for the flower eDNA metabarcoding
    pipeline
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:67aadf77132e1abf4781fdb9752519995e87938036becec143a94ee27dea1518
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - tobiasgf/Bioinformatic-tools
  edam_operation: http://edamontology.org/operation_0232
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# coi-16s-amplicon-bioinformatics

## Summary

Two-marker eDNA metabarcoding bioinformatics for wild-flower arthropod detection: DADA2 MOTU inference on mitochondrial 16S rRNA and COI amplicons, replicate filtering to at least two PCR replicates, and taxonomic assignment against arthropod reference databases with species calls requiring 99-100% sequence identity.

## When to use

Use when raw 16S/COI amplicon FASTQ data from eDNA or metabarcoding samples must be denoised into MOTUs, filtered by PCR-replicate detection, taxonomically assigned, and summarized into read counts and taxonomic richness.

## When NOT to use

- Input is not paired 16S/COI amplicon data from eDNA samples — e.g., shotgun metagenomes or single-marker host-barcode runs where replicate-level MOTU filtering does not apply
- The MOTU table is already taxonomically assigned and replicate-filtered; only the downstream richness/dominance summarization is needed

## Inputs

- DADA2-inferred MOTU table for mitochondrial 16S rRNA and COI markers (MOTUs retained only when detected in at least 2 PCR replicates)
- Raw amplicon FASTQ reads from the two primer sets targeting 16S rRNA and COI
- Arthropod reference sequence databases for taxonomic assignment

## Outputs

- Total arthropod read counts per marker (e.g., 23,517,933 COI reads and 658,159 16S reads)
- Taxonomic richness summary (species, families, orders — e.g., at least 135 species / 67 families / 14 orders)
- Per-species read-share fractions quantifying dominance (e.g., Thrips major at 30% of COI reads; Meligethes planiusculus at 51% of 16S reads)

## How to apply

Load the DADA2-inferred MOTU table for the two marker sets (mitochondrial 16S rRNA and COI), retaining MOTUs only when detected in at least two PCR replicates. Assign taxonomy by matching sequences against arthropod reference databases, applying the paper's rule that species-level assignment requires a 99-100% sequence-identity match. Filter assignments to arthropods and tally total arthropod reads per marker (expected order of magnitude: 23,517,933 COI reads and 658,159 16S reads in the study). Aggregate assigned MOTUs into taxonomic richness counts and compute per-species read-share fractions to quantify dominance; in the article this recovered at least 135 arthropod species in 67 families and 14 orders, with Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads. Validate every computed figure against the published values and flag any marker whose arthropod read total deviates.

## Related tools

- **DADA2** (Denoising and MOTU inference from 16S rRNA and COI amplicon reads prior to replicate filtering) — tobiasgf/Bioinformatic-tools
- **tobiasgf/Bioinformatic-tools** (Repository hosting the bioinformatic scripts used for the flower eDNA metabarcoding pipeline) — tobiasgf/Bioinformatic-tools

## Evaluation signals

- Arthropod read totals per marker match the reported figures: 23,517,933 COI reads and 658,159 16S reads; flag any marker whose total deviates
- Richness counts reproduce 135 arthropod species in 67 families and 14 orders
- Dominance shares reproduce Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads
- Every retained MOTU is detected in at least 2 PCR replicates; MOTUs below this threshold are excluded
- All species-level assignments are supported by a 99-100% sequence-identity match against the arthropod reference databases

## Limitations

- The experiment was not exhaustive — the article notes that an even higher arthropod richness could be obtained using this eDNA approach, so counts are reported as 'at least' 135 species
- Species-level taxonomic assignment depends on reference database coverage and the strict 99-100% identity rule; sequences without such matches cannot be resolved to species

## Evidence

- [abstract] Two generic primers targeting mitochondrial genes: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] Final richness from replicate-filtered data: [evidence span withheld — anchor status: not_found]
- [other] Species assignment identity rule and validation targets: [evidence span withheld — anchor status: not_found]
- [other] Reproducibility check on reported values: [evidence span withheld — anchor status: not_found]
