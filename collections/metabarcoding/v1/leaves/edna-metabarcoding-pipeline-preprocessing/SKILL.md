---
name: edna-metabarcoding-pipeline-preprocessing
description: Use when a metabarcoding pipeline has produced dereplicated amplicon
  sequences (e.g. after obiuniq on quality- and length-filtered paired-end reads)
  and chimeras must be removed before MOTU clustering.
when_to_use_negative:
- Input is raw paired-end FASTQ that has not yet been aligned, quality-filtered, length-filtered
  and dereplicated — uchime-denovo in this pipeline operates on dereplicated reads,
  not raw reads
- A curated reference-based chimera detection strategy is required (uchime-denovo
  is de novo and does not use a reference database)
- Input is already a MOTU table or post-clustering representative sequences — chimera
  filtering belongs upstream of sumaclust clustering
edam_operation: http://edamontology.org/operation_3187
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: vsearch
  role: Implements the uchime-denovo algorithm used to detect and remove chimeric
    sequences de novo from the dereplicated amplicon reads
- name: obiuniq
  role: Dereplicates the quality- and length-filtered reads upstream, producing the
    dereplicated FASTA that is the input to chimera removal
- name: obigrep
  role: Applies the upstream 120–135 bp length filter that defines the read set on
    which chimera removal is performed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:5ca2410600cc1b29fa6e0a4e9d44135be2d24d2a7d7e9c198135e3937e213834
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - obigrep
  - obiuniq
  - vsearch
  edam_operation: http://edamontology.org/operation_3187
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# edna-metabarcoding-pipeline-preprocessing

## Summary

De novo chimera removal step for eDNA metabarcoding amplicon reads: chimeric sequences are detected and discarded using the uchime-denovo algorithm implemented in vsearch, operating on dereplicated reads without a reference database, before MOTU delimitation.

## When to use

Use when a metabarcoding pipeline has produced dereplicated amplicon sequences (e.g. after obiuniq on quality- and length-filtered paired-end reads) and chimeras must be removed before MOTU clustering.

## When NOT to use

- Input is raw paired-end FASTQ that has not yet been aligned, quality-filtered, length-filtered and dereplicated — uchime-denovo in this pipeline operates on dereplicated reads, not raw reads
- A curated reference-based chimera detection strategy is required (uchime-denovo is de novo and does not use a reference database)
- Input is already a MOTU table or post-clustering representative sequences — chimera filtering belongs upstream of sumaclust clustering

## Inputs

- Dereplicated FASTA file of length- and quality-filtered eDNA amplicon reads (produced by obiuniq after obigrep 120–135 bp filtering and removal of reads with ambiguous bases)

## Outputs

- Chimera-filtered FASTA file containing only non-chimeric sequences, ready for MOTU clustering with sumaclust

## How to apply

Position this step after paired-end alignment (illuminapairedend, quality score >40 kept), length filtering to the 120–135 bp target size (obigrep), removal of reads with ambiguous bases, and dereplication with obiuniq. Load the dereplicated FASTA and run uchime-denovo in vsearch in de novo mode, so chimeras are detected purely from abundance and sequence relationships without any reference database — appropriate when a trusted chimera reference set is unavailable. Export only the non-chimeric sequences to a FASTA output file, discarding sequences flagged as chimeric; the chimera-filtered dataset is then carried forward to MOTU delimitation with sumaclust at a 99% similarity threshold. The rationale is that PCR chimeras would otherwise inflate MOTU richness and create false taxonomic assignments.

## Related tools

- **vsearch** (Implements the uchime-denovo algorithm used to detect and remove chimeric sequences de novo from the dereplicated amplicon reads)
- **obiuniq** (Dereplicates the quality- and length-filtered reads upstream, producing the dereplicated FASTA that is the input to chimera removal)
- **obigrep** (Applies the upstream 120–135 bp length filter that defines the read set on which chimera removal is performed)

## Examples

```
vsearch --uchime_denovo dereplicated.fasta --nonchimeras chimera_filtered.fasta
```

## Evaluation signals

- The chimera-filtered output FASTA exists and is a valid FASTA file
- No sequences flagged as chimeric by uchime-denovo remain in the output dataset
- Output contains only non-chimeric dereplicated sequences, i.e. a subset of the input dereplicated FASTA
- The step is executed after dereplication and before sumaclust MOTU delimitation at the 99% similarity threshold, preserving pipeline order
- Downstream MOTU delimitation and ecotag taxonomic assignment run successfully on the chimera-filtered sequences

## Limitations

- uchime-denovo is a de novo method and detects chimeras without a reference database, so detection depends on abundance/sequence signals rather than curated references
- Chimera filtering does not address the acknowledged trade-off between taxonomic resolution and non-target amplification, nor single-read MOTUs that may stem from contamination or sequencing error (handled separately by removing single-read MOTUs)
- Some elasmobranch species may be missed upstream due to primer mismatches (e.g. Ginglymostoma cirratum), which chimera removal cannot recover

## Evidence

- [discussion] a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] a length filter (obigrep) was applied to the aligned reads (120–135 bp): "a length filter (obigrep) was applied to the aligned reads (120–135 bp)"
- [discussion] The bioinformatic analysis was based on the OBITools metabarcoding software suite75.: "The bioinformatic analysis was based on the OBITools metabarcoding software suite75."
- [readme] This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences.: "This is a collection of Reference databases useful for taxonomic assignment of Eukaryotic sequences."
