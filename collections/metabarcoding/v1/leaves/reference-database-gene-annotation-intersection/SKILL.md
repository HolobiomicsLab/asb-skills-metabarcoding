---
name: reference-database-gene-annotation-intersection
description: Use when per-read mappings (with coordinates and mapping qualities) from
  MetaMaps against an annotated reference database need to be converted into functional
  profiles, e.g. COG abundance rankings for a long-read metagenome.
when_to_use_negative:
- Input is a short-read sample or a sample with a high proportion of short reads —
  reads below the minimum length end up with minimizer sets too small to reliably
  determine mapping locations
- The goal is purely taxonomic composition without functional annotation — MetaMaps'
  EM-based WIMP output alone suffices, and gene-interval intersection adds no value
- The reference database genes carry no annotation (no COG/eggNOG-mapper categories),
  so read–gene overlaps cannot be assigned to functional groups
edam_operation: http://edamontology.org/operation_3192
edam_topics:
- http://edamontology.org/topic_3697
- http://edamontology.org/topic_0622
tools:
- name: MetaMaps
  role: Produces the per-read approximate mapping locations, estimated alignment identities,
    and mapping qualities that are intersected with annotated gene intervals; also
    performs EM-based sample composition estimation
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying minimizer-based approximate mapping algorithm used by MetaMaps
    to generate candidate mapping locations for each long read
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:d72694523bf3469ce85826338db3f64b6edf0407797e8520d07bf3fffb1212c7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_3192
  edam_topics:
  - http://edamontology.org/topic_3697
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# reference-database-gene-annotation-intersection

## Summary

Intersects per-read mapping locations from MetaMaps long-read classification with annotated gene intervals (e.g. eggNOG-mapper COG assignments) in the reference database, then aggregates read–gene overlaps into counts per functional category such as COG groups S, K, and E.

## When to use

Use when per-read mappings (with coordinates and mapping qualities) from MetaMaps against an annotated reference database need to be converted into functional profiles, e.g. COG abundance rankings for a long-read metagenome.

## When NOT to use

- Input is a short-read sample or a sample with a high proportion of short reads — reads below the minimum length end up with minimizer sets too small to reliably determine mapping locations
- The goal is purely taxonomic composition without functional annotation — MetaMaps' EM-based WIMP output alone suffices, and gene-interval intersection adds no value
- The reference database genes carry no annotation (no COG/eggNOG-mapper categories), so read–gene overlaps cannot be assigned to functional groups

## Inputs

- Long-read metagenomic FASTQ (e.g. HMP7 PacBio/Oxford Nanopore reads)
- MetaMaps reference database FASTA and extended taxonomy (e.g. 24,945-genome miniSeq+H-style database with eggNOG-mapper COG annotations on genes)
- MetaMaps per-read mapping output (classification_results.EM, classification_results.EM.reads2Taxon)
- Annotated gene interval table (gene coordinates plus eggNOG-mapper COG category per gene)

## Outputs

- Read-to-gene assignment table (read ID, mapped contig coordinates, gene overlap)
- Ranked COG category abundance table (cog_abundance_profile.csv) with counts per functional group
- Per-read mapping locations, estimated identities, and mapping qualities (classification_results.EM)

## How to apply

Run MetaMaps (mapDirectly + classify) so that each read's approximate mapping location, estimated alignment identity, and mapping quality are captured in the classification_results.EM output, where mapping qualities of all mappings for one read sum to 1. Extract per-read mapped genomic coordinates for all classified reads, then intersect each read's mapped interval with the annotated gene intervals in the reference database to assign reads to genes and their eggNOG-mapper COG categories. Aggregate the read–gene overlaps into counts per COG functional category (e.g., S = Function unknown, K = Transcription, E = Amino acid transport and metabolism) and rank categories by abundance. Correct application is judged by reproducing the expected ordering of COG categories (S most represented, then K and E) and by checking that mapping qualities across each read's mappings sum to 1 before collapsing to maximum-likelihood assignments.

## Related tools

- **MetaMaps** (Produces the per-read approximate mapping locations, estimated alignment identities, and mapping qualities that are intersected with annotated gene intervals; also performs EM-based sample composition estimation) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying minimizer-based approximate mapping algorithm used by MetaMaps to generate candidate mapping locations for each long read) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- For every read, the mapping qualities of all its mappings in classification_results.EM sum to 1
- Every mapped read interval overlaps exactly one assigned gene interval; reads with no gene overlap are excluded from COG counts
- Every COG category label in cog_abundance_profile.csv corresponds to an eggNOG-mapper annotation present in the database gene table
- Ranking check: category S (Function unknown) is most abundant, followed by K (Transcription) and E (Amino acid transport and metabolism), matching the reported Supplementary Fig. 2 ordering
- classification_results.EM.reads2Taxon contains exactly one assigned taxon per classified read, with taxon IDs interpretable via the extended database taxonomy

## Limitations

- Reads that fall below the chosen minimum length end up with minimizer sets too small to reliably determine their mapping locations, so they remain unassigned and cannot be assigned to genes
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- Database–sample incongruity (uncharacterized taxonomic entities) can distort which reference genes/COGs reads are attributed to; complex samples comprising many non-characterized entities remain challenging

## Evidence

- [abstract] MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence): "MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence)"
- [intro] First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29.: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [discussion] Reads below minimum length remain unassigned due to small minimizer sets: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
- [readme] MetaMaps alignment carries approximate alignment location, estimated identity and mapping quality: "each MetaMaps alignment comes with an approximate alignment location, an estimated alignment identity and a mapping quality."
- [readme] The EM output file contains the complete set of approximate read mappings: [evidence span withheld — anchor status: not_found]
