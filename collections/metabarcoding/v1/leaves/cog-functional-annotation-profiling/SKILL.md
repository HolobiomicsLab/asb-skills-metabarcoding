---
name: cog-functional-annotation-profiling
description: Use when long reads (e.g., HMP7 ONT/PacBio data) have been mapped with
  MetaMaps against a COG-annotated reference database and the goal is to rank functional
  categories (COG groups like S, K, E) by the number of reads overlapping annotated
  gene intervals.
when_to_use_negative:
- The sample has a high proportion of short reads — reads below the minimum length
  end up with minimizer sets too small for reliable mapping and remain unassigned
- Only taxonomic composition is needed (use the MetaMaps EM/WIMP classification output
  directly instead of gene-overlap profiling)
- The reference database lacks gene interval and COG/eggNOG-mapper annotation, so
  reads cannot be assigned to functional categories
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_0622
tools:
- name: MetaMaps
  role: Maps long reads to the COG-annotated database (minimizer-based approximate
    mapping + probabilistic scoring) and outputs per-read mapping locations and mapping
    qualities used for gene-overlap counting
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying approximate mapping algorithm used by MetaMaps to generate candidate
    mapping locations
  repo: https://github.com/marbl/MashMap
- name: eggNOG-mapper
  role: Provides the COG group assignments annotating the genes of the reference database
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:2d565cace06899b284193fe568c7a1d7f13aefb6b5621c82aada3d7974123cec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  - eggNOG-mapper
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cog-functional-annotation-profiling

## Summary

Profiles COG functional-category abundance from long-read metagenomic mappings by intersecting MetaMaps per-read mapping locations against eggNOG-mapper COG-annotated gene intervals in the reference database, then ranking categories by overlapping-read counts (e.g., HMP7: S > K > E).

## When to use

Use when long reads (e.g., HMP7 ONT/PacBio data) have been mapped with MetaMaps against a COG-annotated reference database and the goal is to rank functional categories (COG groups like S, K, E) by the number of reads overlapping annotated gene intervals.

## When NOT to use

- The sample has a high proportion of short reads — reads below the minimum length end up with minimizer sets too small for reliable mapping and remain unassigned
- Only taxonomic composition is needed (use the MetaMaps EM/WIMP classification output directly instead of gene-overlap profiling)
- The reference database lacks gene interval and COG/eggNOG-mapper annotation, so reads cannot be assigned to functional categories

## Inputs

- Long-read metagenomic FASTQ (e.g., HMP7 dataset)
- MetaMaps reference database with gene-level eggNOG-mapper COG annotations (24,945-genome extended DB)
- Annotated gene interval table (gene coordinates mapped to COG categories)

## Outputs

- cog_abundance_profile.csv (COG category counts, ranked by abundance)
- Ranked COG category abundance profile (e.g., S > K > E)

## How to apply

Run MetaMaps (mapDirectly followed by classify) on the long-read FASTQ against the extended 24,945-genome reference database whose genes carry eggNOG-mapper COG annotations; this produces classification_results.EM with per-mapping contig, start/stop coordinates and mapping quality. Extract the per-read mapping locations for all classified reads, then intersect each read's mapped genomic coordinates with the annotated gene intervals to assign reads to genes and their COG categories. Aggregate the read-gene overlaps into counts per COG functional category (e.g., S = Function unknown, K = Transcription, E = Amino acid transport and metabolism) and rank categories by abundance. Rationale: MetaMaps' minimizer-based approximate mapping (built on MashMap) plus probabilistic scoring yields mapping locations and qualities for all classified reads, which is exactly what enables this gene-overlap functional analysis; validate the ranked profile against the reported Supplementary Fig. 2 ordering and write results to cog_abundance_profile.csv.

## Related tools

- **MetaMaps** (Maps long reads to the COG-annotated database (minimizer-based approximate mapping + probabilistic scoring) and outputs per-read mapping locations and mapping qualities used for gene-overlap counting) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying approximate mapping algorithm used by MetaMaps to generate candidate mapping locations) — https://github.com/marbl/MashMap
- **eggNOG-mapper** (Provides the COG group assignments annotating the genes of the reference database)

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q HMP7.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Ranked abundance profile reproduces the reported ordering: category S (Function unknown) most abundant, followed by K (Transcription) and E (Amino acid transport and metabolism) as in Supplementary Fig. 2
- Every read assignment traces to a mapping in classification_results.EM with coordinates, estimated identity, and a mapping quality (mapping qualities of all mappings for one read sum to 1)
- Counts per COG category equal the number of classified reads whose mapped intervals overlap COG-annotated gene coordinates; no double-counting of a single read across categories unless it overlaps multiple genes
- cog_abundance_profile.csv contains one row per COG category with a count and is sorted in descending abundance order

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- MetaMaps outputs approximate (not exact) alignment locations, so read-gene overlap calls carry positional uncertainty at gene boundaries
- Database-sample incongruity (uncharacterized taxonomic entities) can degrade mapping fidelity and thus the functional profile

## Evidence

- [abstract] MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence): "MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence) and detection of incongruities between sample and reference genomes"
- [intro] Minimizer-based mapping generates candidate locations per read: "First, a list of possible mapping locations for each long read is generated using a minimizer-based approximate mapping strategy29."
- [readme] Each alignment carries location, identity, and mapping quality: "For example, each MetaMaps alignment comes with an approximate alignment location, an estimated alignment identity and a mapping quality."
- [readme] EM output file contains per-mapping coordinates and quality fields: [evidence span withheld — anchor status: not_found]
