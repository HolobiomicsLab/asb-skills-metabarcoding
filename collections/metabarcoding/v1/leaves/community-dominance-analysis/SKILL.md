---
name: community-dominance-analysis
description: Use when a DADA2-inferred, replicate-filtered MOTU table has been taxonomically
  assigned and you need per-species, per-family, or per-order read fractions to characterize
  community dominance and compare marker sets.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet gone through DADA2 MOTU inference and
  replicate filtering
- You need presence/absence occupancy patterns or community clustering by plant species
  rather than quantitative read-share dominance
- MOTU taxonomy has not been assigned, so reads cannot be attributed to arthropod
  species
edam_operation: http://edamontology.org/operation_2940
edam_topics:
- http://edamontology.org/topic_0637
tools:
- name: DADA2
  role: Infers MOTUs from the two primer sets (mitochondrial 16S rRNA and COI); its
    filtered MOTU table is the input read matrix for dominance fractions
  repo: tobiasgf/Bioinformatic-tools
- name: tobiasgf/Bioinformatic-tools
  role: Repository of bioinformatic scripts used to process the flower-eDNA reads,
    including taxonomy assignment of MOTUs prior to dominance summarization
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:56c540926e8426fd32449105dc54ac631f95f564f7384028787a70245b45f105
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - DADA2
  - tobiasgf/Bioinformatic-tools
  edam_operation: http://edamontology.org/operation_2940
  edam_topics:
  - http://edamontology.org/topic_0637
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# community-dominance-analysis

## Summary

Quantifies per-taxon read-share fractions in an eDNA metabarcoding dataset to identify which species dominate the community, e.g. Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads in the flower-eDNA study.

## When to use

Use when a DADA2-inferred, replicate-filtered MOTU table has been taxonomically assigned and you need per-species, per-family, or per-order read fractions to characterize community dominance and compare marker sets.

## When NOT to use

- Input is raw FASTQ reads that have not yet gone through DADA2 MOTU inference and replicate filtering
- You need presence/absence occupancy patterns or community clustering by plant species rather than quantitative read-share dominance
- MOTU taxonomy has not been assigned, so reads cannot be attributed to arthropod species

## Inputs

- DADA2-inferred MOTU count table filtered to MOTUs detected in at least 2 PCR replicates
- Taxonomy assignment table for each MOTU (species-level assignments at 99-100% sequence identity against arthropod reference databases)
- Marker-set labels distinguishing mitochondrial 16S rRNA and COI amplicons

## Outputs

- Per-species read-share fractions per marker (e.g. Thrips major 30% of COI reads, Meligethes planiusculus 51% of 16S reads)
- Total arthropod read counts per marker (23,517,933 COI; 658,159 16S)
- Taxonomic richness summary (species, family, order counts)

## How to apply

Start from the DADA2 MOTU table retained only when detected in at least 2 PCR replicates, with taxonomy assigned by matching sequences against arthropod reference databases under the rule that species-level assignment requires a 99-100% sequence-identity match. Sum reads per assigned MOTU and filter to arthropods, keeping the two marker sets separate (mitochondrial 16S rRNA and COI) because dominance differs by marker. Aggregate assigned MOTUs per species and divide by the marker's total arthropod reads (23,517,933 COI reads and 658,159 16S reads in the study) to obtain read-share fractions. Rank species by fraction to report dominance, then cross-check richness context (135 species in 67 families and 14 orders). Flag any marker whose arthropod read total or dominant-taxon share deviates from the published values.

## Related tools

- **DADA2** (Infers MOTUs from the two primer sets (mitochondrial 16S rRNA and COI); its filtered MOTU table is the input read matrix for dominance fractions) — tobiasgf/Bioinformatic-tools
- **tobiasgf/Bioinformatic-tools** (Repository of bioinformatic scripts used to process the flower-eDNA reads, including taxonomy assignment of MOTUs prior to dominance summarization) — tobiasgf/Bioinformatic-tools

## Examples

```
shares = reads.groupby(['marker','species']).reads.sum().sort_values(ascending=False) / reads.groupby('marker').reads.transform('sum')
```

## Evaluation signals

- Total arthropod reads per marker reproduce the reported figures: 23,517,933 COI reads and 658,159 16S reads
- Richness tally matches at least 135 arthropod species in 67 families and 14 orders
- Dominant-species shares match Thrips major at 30% of COI reads and Meligethes planiusculus at 51% of 16S reads
- Per-species read shares sum to 1 (or 100%) within each marker set
- Only MOTUs detected in at least 2 PCR replicates contribute to the dominance estimates

## Limitations

- Study reports 'at least' 135 species: the experiment was not exhaustive and higher arthropod richness could be obtained with this eDNA approach
- Dominance estimates are marker-dependent; the dominant taxon differs between COI (Thrips major) and 16S (Meligethes planiusculus)
- Species-level shares depend on the 99-100% sequence-identity assignment rule; MOTUs without such matches cannot be attributed to species

## Evidence

- [abstract] Richness and dominance context: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups"
- [abstract] Marker sets define the two read-share denominators: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [other] Dominance shares reproduced by this skill: [evidence span withheld — anchor status: not_found]
- [other] Replicate filter that must precede dominance analysis: [evidence span withheld — anchor status: not_found]
