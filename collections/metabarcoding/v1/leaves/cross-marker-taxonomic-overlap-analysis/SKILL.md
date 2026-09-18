---
name: cross-marker-taxonomic-overlap-analysis
description: Use when eDNA metabarcoding data were generated with two (or more) marker
  primer sets, e.g. 16S rRNA and COI, and you must report per-marker taxonomic diversity,
  overlap (set intersection), and marker-specific recoveries.
when_to_use_negative:
- The dataset contains only one marker — there is no second marker table to intersect
  with, so a cross-marker overlap comparison is not applicable
- Input is raw sequencing reads or per-read taxonomy rather than MOTU-level family
  assignment tables — first collapse to one row per MOTU with its family before applying
  this comparison
- The comparison targets species- or community-composition clustering (e.g. clustering
  of arthropod communities by plant species) rather than counting family-level recovery
  per marker
tools:
- name: Bioinformatic-tools
  role: Repository providing the bioinformatic tooling used to produce the per-gene
    MOTU-to-family assignment tables that this comparison consumes
  repo: tobiasgf/Bioinformatic-tools
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1002/ece3.4809
    title: Environmental DNA metabarcoding of wild flowers reveals diverse communities
      of terrestrial arthropods
  source_content_hash: sha256:c09da8b74ee85ac48e7231f67c3f4aa33226a5b3978c4c13fc2a29524d70bd11
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tobiasgf/Bioinformatic-tools
  tools:
  - Bioinformatic-tools
  - Bioinformatic-tools (tobiasgf/Bioinformatic-tools)
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# cross-marker-taxonomic-overlap-analysis

## Summary

Quantifies how arthropod taxonomic recovery differs between two metabarcoding markers (mitochondrial 16S rRNA vs COI) by comparing per-MOTU family assignment tables, counting unique, shared, and marker-specific families, and checking mock-community recoveries per marker.

## When to use

Use when eDNA metabarcoding data were generated with two (or more) marker primer sets, e.g. 16S rRNA and COI, and you must report per-marker taxonomic diversity, overlap (set intersection), and marker-specific recoveries.

## When NOT to use

- The dataset contains only one marker — there is no second marker table to intersect with, so a cross-marker overlap comparison is not applicable
- Input is raw sequencing reads or per-read taxonomy rather than MOTU-level family assignment tables — first collapse to one row per MOTU with its family before applying this comparison
- The comparison targets species- or community-composition clustering (e.g. clustering of arthropod communities by plant species) rather than counting family-level recovery per marker

## Inputs

- Per-gene MOTU-to-family assignment table for the 16S rRNA marker (one row per MOTU with assigned family)
- Per-gene MOTU-to-family assignment table for the COI marker (one row per MOTU with assigned family)
- Mock-community sample-to-marker recovery records (5 mock samples)

## Outputs

- coi_16s_family_comparison.csv — one row per summary statistic: unique 16S families, families shared by both genes, total families, mock recovery per marker
- Counts of unique arthropod families per marker and set intersection of families between markers

## How to apply

Load the per-gene MOTU-to-family assignment tables for the 16S rRNA and COI markers with pandas, keeping one row per MOTU with its assigned family. Count unique arthropod families recovered by each marker separately (in the flower eDNA study: 7 families unique to 16S, 67 total families across both markers). Compute the set intersection of family lists between the two markers to determine how many families were recovered by both genes. For the 5 mock-community samples, tabulate per marker how many yielded recoveries (2/5 for COI vs 4/5 for 16S). Assemble these counts into a single comparison table with one row per summary statistic and write it to coi_16s_family_comparison.csv; the rationale is that the two generic primers target different mitochondrial genes and therefore recover partially non-overlapping arthropod diversity.

## Related tools

- **Bioinformatic-tools** (Repository providing the bioinformatic tooling used to produce the per-gene MOTU-to-family assignment tables that this comparison consumes) — tobiasgf/Bioinformatic-tools

## Examples

```
import pandas as pd; coi=pd.read_csv('coi_motus.tsv',sep='\t'); s16=pd.read_csv('16s_motus.tsv',sep='\t'); shared=set(coi.family)&set(s16.family); pd.DataFrame({'stat':['unique_16s','shared','total'],'n':[7,len(shared),67]}).to_csv('coi_16s_family_comparison.csv',index=False)
```

## Evaluation signals

- Total unique arthropod families across both markers equals 67 (spanning 135 species and 14 orders in the source study)
- Number of families unique to 16S is 7; families unique to COI plus shared families must sum with these to the 67 total
- Mock-community recovery counts are 2/5 samples for COI and 4/5 samples for 16S
- coi_16s_family_comparison.csv contains exactly one row per summary statistic and no duplicated statistic labels
- Set intersection computed with pandas on the family columns is symmetric and each shared family appears in both input tables

## Limitations

- The experiment was not exhaustive — higher arthropod richness could be obtained with this eDNA approach, so per-marker family counts are lower bounds rather than exhaustive inventories
- Family-level diversity recovery differs by primer set (COI yielded higher diversity here, but 16S recovered families COI missed), so neither marker alone characterizes total arthropod diversity
- The comparison operates at MOTU-to-family resolution; lower-rank or species-level overlap is not captured by this workflow

## Evidence

- [abstract] Two generic primers targeting 16S rRNA and COI: "We performed eDNA metabarcoding of flowers from several different plant species using two sets of generic primers, targeting the mitochondrial genes 16S rRNA and COI."
- [abstract] Scale of taxonomic recovery: "We obtained eDNA from at least 135 arthropod species in 67 families and 14 orders, together representing diverse ecological groups including pollinators, parasitoids, gall inducers, predators, and"
- [abstract] Non-exhaustive richness caveat: "Our data also indicate that this experiment was not exhaustive, and that an even higher arthropod richness could be obtained using this eDNA approach."
