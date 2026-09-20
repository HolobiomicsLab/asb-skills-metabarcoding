---
name: metagenomic-abundance-profile-visualization
description: Use when MetaMaps per-read mappings (classification_results.EM) from
  long reads (e.g., HMP7) must be intersected with annotated gene intervals and aggregated
  into a ranked abundance profile such as COG category counts.
when_to_use_negative:
- Sample has a high proportion of short reads — MetaMaps does not handle short reads
  well and sub-minimum-length reads end up unassigned
- Input is already a quantified abundance or feature table rather than raw long reads
  needing taxonomic/functional assignment
- Only overall sample composition is needed at taxon level — the classification_results.EM.WIMP
  output already provides Absolute/EMFrequency/PotFrequency without gene-level intersection
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_3697
tools:
- name: MetaMaps
  role: Produces the per-read mapping locations, estimated alignment identities, and
    mapping qualities that are aggregated into the abundance profile
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Minimizer-based approximate mapping algorithm underlying MetaMaps' generation
    of candidate mapping locations
  repo: https://github.com/marbl/MashMap
- name: eggNOG-mapper
  role: Provides COG group annotations on reference database genes used to translate
    read-gene overlaps into functional categories
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:ddfb0a9d97069920f6c111c423ff9867144fc07ae7ca13fbc5ac573091044252
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  - eggNOG-mapper
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_3697
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# metagenomic-abundance-profile-visualization

## Summary

Build and rank abundance profiles (e.g., per-COG functional category counts) from MetaMaps per-read mapping output on long-read metagenomic data, then check the ranking against the published ordering such as Supplementary Fig. 2 (S > K > E).

## When to use

Use when MetaMaps per-read mappings (classification_results.EM) from long reads (e.g., HMP7) must be intersected with annotated gene intervals and aggregated into a ranked abundance profile such as COG category counts.

## When NOT to use

- Sample has a high proportion of short reads — MetaMaps does not handle short reads well and sub-minimum-length reads end up unassigned
- Input is already a quantified abundance or feature table rather than raw long reads needing taxonomic/functional assignment
- Only overall sample composition is needed at taxon level — the classification_results.EM.WIMP output already provides Absolute/EMFrequency/PotFrequency without gene-level intersection

## Inputs

- Long-read metagenomic FASTQ (e.g., HMP7 Oxford Nanopore reads)
- MetaMaps reference database FASTA and taxonomy (e.g., extended 24,945-genome database)
- Annotated gene intervals with eggNOG-mapper COG category assignments
- MetaMaps output files: classification_results.EM, classification_results.EM.reads2Taxon

## Outputs

- Ranked COG abundance profile (cog_abundance_profile.csv) with counts per functional category (e.g., S, K, E)
- Per-read gene/COG assignments from coordinate intersection

## How to apply

Run MetaMaps in two steps — 'mapDirectly' followed by 'classify' — against the extended reference database (e.g., 24,945 genomes whose genes carry eggNOG-mapper COG annotations), using minimizer-based approximate mapping and probabilistic scoring. Extract per-read mapping locations, estimated alignment identities, and mapping qualities for all classified reads from classification_results.EM and classification_results.EM.reads2Taxon. Intersect each read's mapped contig coordinates with the annotated gene intervals to assign reads to genes and their COG categories. Aggregate read-gene overlaps into counts per COG functional category (e.g., S, K, E), rank categories by abundance, and write cog_abundance_profile.csv. Judge correctness by comparing the ranked profile to the reported Supplementary Fig. 2 ordering — category S (Function unknown) most abundant, followed by K (Transcription) and E (Amino acid transport and metabolism).

## Related tools

- **MetaMaps** (Produces the per-read mapping locations, estimated alignment identities, and mapping qualities that are aggregated into the abundance profile) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Minimizer-based approximate mapping algorithm underlying MetaMaps' generation of candidate mapping locations) — https://github.com/marbl/MashMap
- **eggNOG-mapper** (Provides COG group annotations on reference database genes used to translate read-gene overlaps into functional categories)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- cog_abundance_profile.csv exists with categories ranked so S (Function unknown) is most abundant, followed by K (Transcription) and E (Amino acid transport and metabolism), matching Supplementary Fig. 2
- Every read's mapping qualities in classification_results.EM sum to 1 (README invariant)
- Each line of classification_results.EM contains the 14 documented fields (read ID, coordinates, strand, contig ID, identities, MinHash intersection/union, mapping quality)
- All classified reads in classification_results.EM.reads2Taxon carry a taxon ID interpretable via the database taxonomy sub-directory
- Read-gene overlap counts are non-zero for dominated categories and counts increase monotonically across the ranked profile

## Limitations

- Reads below the chosen minimum length have minimizer sets too small to reliably determine mapping locations and remain unassigned
- Analysis of samples with a high proportion of short reads is not well addressed by MetaMaps
- Complex samples comprising many non-characterized entities remain challenging, and database-sample incongruity (yet-uncharacterized taxonomic entities) is an unresolved concern

## Evidence

- [abstract] MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence): "MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities"
- [intro] minimizer-based mapping generates candidate locations: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [readme] README describes per-mapping output fields used for aggregation: [evidence span withheld — anchor status: not_found]
- [discussion] short/sub-minimum reads fail mapping: "Reads that fall below the chosen minimum length end up with minimizer sets that are too small to reliably determine their mapping locations."
