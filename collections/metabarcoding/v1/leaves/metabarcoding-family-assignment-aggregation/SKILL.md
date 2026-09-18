---
name: metabarcoding-family-assignment-aggregation
description: Use when per-MOTU metabarcoding results must be collapsed to arthropod
  family level and compared across primer sets (e.g. mitochondrial 16S rRNA vs COI)
  or summarized per plant species/mock sample.
when_to_use_negative:
- Input is raw sequencing reads rather than already-generated MOTU-to-family assignment
  tables — MOTU inference and taxonomic assignment must precede this skill
- The goal is species-level ecological interpretation rather than family-level diversity
  aggregation (the 135 species were collapsed into 67 families here)
- The study system has no multi-marker design or mock community, making the COI vs
  16S comparison statistics inapplicable
edam_operation: http://edamontology.org/operation_0226
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: tobiasgf/Bioinformatic-tools
  role: Bioinformatic scripts used to generate the per-gene MOTU-to-family assignment
    tables that this skill aggregates and compares
  repo: tobiasgf/Bioinformatic-tools
- name: bipartite (R package)
  role: frame2webs / plotweb / visweb functions used to construct and draw the plant-species
    x arthropod-family bipartite network (Figure 6)
- name: pandas
  role: Loads and tabulates the per-gene MOTU-to-family assignment tables and assembles
    the marker-comparison CSV
merged_aliases:
- metabarcoding-family-diversity-summarization
merged_alias_records:
- alias: metabarcoding-family-diversity-summarization
  slug: metabarcoding-family-diversity-summarization
  cosine_score: 0.8419
  method: embedding-cosine
  decision: needs_review
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:fa1c264a604246f372549c3811e42f8bc36b0788f3aff5dbb0b0fa41276670f9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - Bioinformatic-tools (tobiasgf/Bioinformatic-tools)
  - R
  - bipartite (R package)
  - pandas
  - tobiasgf/Bioinformatic-tools
  edam_operation: http://edamontology.org/operation_0226
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# metabarcoding-family-assignment-aggregation

## Summary

Aggregates per-MOTU metabarcoding taxonomic assignments to arthropod family level and summarizes diversity across markers (16S rRNA vs COI) and plant-sample groups, following the wild-flower eDNA metabarcoding workflow of Thomsen & Sigsgaard (2019), where 135 arthropod species in 67 families and 14 orders were recovered.

## When to use

Use when per-MOTU metabarcoding results must be collapsed to arthropod family level and compared across primer sets (e.g. mitochondrial 16S rRNA vs COI) or summarized per plant species/mock sample.

## When NOT to use

- Input is raw sequencing reads rather than already-generated MOTU-to-family assignment tables — MOTU inference and taxonomic assignment must precede this skill
- The goal is species-level ecological interpretation rather than family-level diversity aggregation (the 135 species were collapsed into 67 families here)
- The study system has no multi-marker design or mock community, making the COI vs 16S comparison statistics inapplicable

## Inputs

- Per-MOTU family assignment tables for the 16S rRNA marker (one row per MOTU with assigned family)
- Per-MOTU family assignment tables for the COI marker (one row per MOTU with assigned family)
- Mock-community sample assignment tables (5 mock samples per marker)
- Plant-species x arthropod-family detection/abundance matrix (7 plant taxa)

## Outputs

- coi_16s_family_comparison.csv (one row per summary statistic: unique 16S families, families shared by both genes, total families, mock recovery per marker)
- figure6_bipartite_network.png (plant-species x arthropod-family bipartite web from the R bipartite package)
- Family-level richness summaries per plant species and per marker

## How to apply

Load the per-gene MOTU-to-family assignment tables (one row per MOTU with its assigned family) for the 16S rRNA and COI markers with pandas. Count unique arthropod families per marker, compute the set intersection to determine how many of the 67 total families are shared, and identify families unique to one marker (7 families were unique to 16S in this study). For mock-community samples, tabulate per marker how many of the 5 mock samples yielded recoveries (COI 2/5 vs 16S 4/5), then assemble a single comparison CSV with one row per summary statistic. For the community-composition view, build a plant-species x arthropod-family detection matrix and render a bipartite network with the R bipartite package (frame2webs / plotweb or visweb), ordering webs so that high-diversity Apiaceae umbels (Angelica, Daucus) and low-diversity single flowers (Echium) are visually apparent as in Figure 6 of the article.

## Related tools

- **tobiasgf/Bioinformatic-tools** (Bioinformatic scripts used to generate the per-gene MOTU-to-family assignment tables that this skill aggregates and compares) — tobiasgf/Bioinformatic-tools
- **bipartite (R package)** (frame2webs / plotweb / visweb functions used to construct and draw the plant-species x arthropod-family bipartite network (Figure 6))
- **pandas** (Loads and tabulates the per-gene MOTU-to-family assignment tables and assembles the marker-comparison CSV)

## Examples

```
import pandas as pd; t16s = pd.read_table('16S_motu_family_assignments.tsv'); tcoi = pd.read_table('COI_motu_family_assignments.tsv'); fams = {g: set(t['family']) for g, t in [('16S', t16s), ('COI', tcoi)]}
```

## Evaluation signals

- Total aggregated diversity matches the published scale: at least 135 arthropod species, 67 families, and 14 orders across the seven plant species
- COI recovers higher arthropod family diversity than 16S in the flower eDNA results, with exactly 7 families unique to 16S
- Mock-community recovery counts reproduce 2/5 samples for COI vs 4/5 for 16S from the assignment tables
- The comparison CSV contains one row per summary statistic (unique 16S families, shared families, total families, mock recovery per marker)
- The bipartite figure is a valid PNG whose diversity ordering matches the published pattern: Apiaceae umbels (Angelica, Daucus) highest and Echium lowest

## Limitations

- The experiment was not exhaustive; the authors state that an even higher arthropod richness could be obtained using this eDNA approach, so family counts are lower bounds
- Marker choice biases recovery: COI yielded higher arthropod diversity overall but recovered fewer mock samples (2/5) than 16S (4/5), so per-marker recovery is not interchangeable
- Family-level aggregation discards species-level resolution of the MOTU assignments

## Evidence

- [abstract] We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI.: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] Arthropod communities clustered together according to plant species.: "Arthropod communities clustered together according to plant species."
- [abstract] Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach.: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
