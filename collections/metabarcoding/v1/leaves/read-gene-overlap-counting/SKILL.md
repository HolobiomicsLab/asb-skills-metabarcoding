---
name: read-gene-overlap-counting
description: Use when you have MetaMaps per-read mappings of long reads (e.g., HMP7
  PacBio/ONT data) against an annotated reference database and need read counts per
  gene or COG functional category.
when_to_use_negative:
- The sample has a high proportion of short reads — reads below the minimum length
  get minimizer sets too small for reliable mapping and end up unassigned
- The goal is taxonomic composition alone (the WIMP/EM output already gives sample
  composition; gene-level intersection is unnecessary)
- The reference database genes lack functional annotations (no COG/gene intervals
  to intersect against)
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: MetaMaps
  role: Produces the per-read mapping locations, estimated alignment identities, and
    mapping qualities that are intersected with annotated gene intervals
  repo: https://github.com/DiltheyLab/MetaMaps
- name: MashMap
  role: Underlying minimizer/minmer-based approximate mapping algorithm used by MetaMaps
    to generate candidate mapping locations
  repo: https://github.com/marbl/MashMap
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-019-10934-2
    title: Strain-level metagenomic assignment and compositional estimation for long
      reads with MetaMaps
  source_content_hash: sha256:a591e72792ce8f578db31ce0c6d9f83ac8d79beacabcb3f90b735c20afec1f97
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DiltheyLab/MetaMaps
  tools:
  - MashMap
  - MetaMaps
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# Read-gene overlap counting

## Summary

Intersects per-read mapping coordinates from MetaMaps long-read metagenomic classification with annotated gene intervals (e.g., eggNOG-mapper COG categories) to aggregate read counts per gene and rank functional categories by abundance.

## When to use

Use when you have MetaMaps per-read mappings of long reads (e.g., HMP7 PacBio/ONT data) against an annotated reference database and need read counts per gene or COG functional category.

## When NOT to use

- The sample has a high proportion of short reads — reads below the minimum length get minimizer sets too small for reliable mapping and end up unassigned
- The goal is taxonomic composition alone (the WIMP/EM output already gives sample composition; gene-level intersection is unnecessary)
- The reference database genes lack functional annotations (no COG/gene intervals to intersect against)

## Inputs

- Long-read FASTQ metagenomic dataset (e.g., HMP7 nanopore reads)
- MetaMaps reference database with annotated genes (extended 24,945-genome build with eggNOG-mapper COG gene assignments)
- classification_results.EM per-read mapping file (read ID, coordinates, strand, contig, identity, mapping quality)

## Outputs

- Read-to-gene overlap assignments with COG category labels
- Ranked per-COG-category abundance counts (e.g., cog_abundance_profile.csv)

## How to apply

Run MetaMaps in two steps (mapDirectly, then classify) against the extended reference database — e.g., the 24,945-genome build whose genes carry eggNOG-mapper COG annotations — using minimizer-based approximate mapping (MashMap-derived) followed by probabilistic scoring. Parse the classification_results.EM output file, which lists, per mapping: read ID, read/contig coordinates, strand, contig ID, estimated alignment identity, and mapping quality (all mappings for a read sum to 1). Extract the mappings for all classified reads and intersect each read's mapped genomic coordinates with the annotated gene intervals in the reference database FASTA to assign reads to genes and their COG categories. Aggregate overlaps into counts per functional category (e.g., S = Function unknown, K = Transcription, E = Amino acid transport and metabolism), rank categories by abundance, and write the ranked profile (e.g., cog_abundance_profile.csv). This is possible because MetaMaps — unlike k-mer classifiers — outputs mapping locations and qualities for all classified reads rather than only taxon labels.

## Related tools

- **MetaMaps** (Produces the per-read mapping locations, estimated alignment identities, and mapping qualities that are intersected with annotated gene intervals) — https://github.com/DiltheyLab/MetaMaps
- **MashMap** (Underlying minimizer/minmer-based approximate mapping algorithm used by MetaMaps to generate candidate mapping locations) — https://github.com/marbl/MashMap

## Examples

```
./metamaps mapDirectly --all -r databases/miniSeq+H/DB.fa -q input.fastq -o classification_results; ./metamaps classify --mappings classification_results --DB databases/miniSeq+H
```

## Evaluation signals

- Every classified read in classification_results.EM has well-formed coordinates (mapping qualities for all mappings of one read sum to 1)
- The ranked COG profile reproduces the reported ordering: category S (Function unknown) most abundant, followed by K (Transcription) and E (Amino acid transport and metabolism)
- All read-gene overlaps fall within the annotated gene intervals of the database contigs (no out-of-bounds coordinates)
- Counts per COG category sum to the number of classified reads assigned to annotated genes (accounting for multi-mapping reads)

## Limitations

- Reads below the chosen minimum length end up with minimizer sets too small to reliably determine mapping locations and remain unassigned
- Database-sample incongruity (uncharacterized taxonomic entities) means reads from novel organisms may fail to map to annotated genes
- MetaMaps does not explicitly model genomic divergence, so novel strains' gene content may be misassigned to close database relatives

## Evidence

- [abstract] MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence): "MetaMaps outputs mapping locations and qualities for all classified reads, enabling functional studies (e.g. gene presence/absence)"
- [full_text] Aggregate the read-gene overlaps into counts per COG functional category (e.g., S, K, E) and rank categories by abundance.: [evidence span withheld — anchor status: not_found]
- [intro] each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm: "each mapping location is scored probabilistically using a model developed here, and total sample composition is estimated using the EM algorithm"
- [readme] each MetaMaps alignment comes with an approximate alignment location, an estimated alignment identity and a mapping quality: "each MetaMaps alignment comes with an approximate alignment location, an estimated alignment identity and a mapping quality"
