---
name: metabarcoding-pipeline-intermediate-file-handling
description: Use when an OBITools-based eDNA metabarcoding workflow has produced filtered,
  aligned reads (e.g. post illuminapairedend, ngsfilter and obigrep 120–135 bp) and
  needs dereplication into a representative-sequence file consumed by vsearch/sumaclust
  steps.
when_to_use_negative:
- Input is raw MiSeq paired-end data that has not yet been aligned (illuminapairedend),
  demultiplexed (ngsfilter) and length-filtered (obigrep 120–135 bp) — dereplication
  must come after those filters
- Input is already a dereplicated representative-sequence file heading to chimera
  removal or MOTU clustering — apply vsearch uchime-denovo or sumaclust instead of
  dereplicating again
- The goal is taxonomic assignment (ecotag) or in silico PCR primer/probe testing
  (ecoPCR) — those steps consume the post-clustering MOTU representative sequences,
  not this intermediate
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: obiuniq
  role: 'Dereplicates the filtered, aligned reads: collapses identical sequences into
    single records and aggregates per-sample read counts into the count annotation'
- name: OBITools
  role: Metabarcoding software suite providing the annotated-FASTQ format and the
    obiuniq command used for this intermediate-file step
- name: vsearch
  role: 'Downstream consumer of the dereplicated file: runs uchime-denovo chimera
    removal on the representative sequences'
- name: sumaclust
  role: 'Downstream consumer of the dereplicated file: delimits MOTUs at a constant
    99% similarity threshold'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:eff837f6723908401fa99ed188b83e12a62bb6029e8d4442a494269dea8d4730
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - obiuniq
  - sumaclust
  - vsearch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# metabarcoding-pipeline-intermediate-file-handling

## Summary

Handles the intermediate dereplicated read file produced after quality filtering, paired-end alignment and length filtering in an OBITools eDNA metabarcoding pipeline: collapsing identical sequences with obiuniq into an OBITools-annotated FASTQ that carries aggregated per-sample counts, ready for downstream chimera removal and MOTU clustering.

## When to use

Use when an OBITools-based eDNA metabarcoding workflow has produced filtered, aligned reads (e.g. post illuminapairedend, ngsfilter and obigrep 120–135 bp) and needs dereplication into a representative-sequence file consumed by vsearch/sumaclust steps.

## When NOT to use

- Input is raw MiSeq paired-end data that has not yet been aligned (illuminapairedend), demultiplexed (ngsfilter) and length-filtered (obigrep 120–135 bp) — dereplication must come after those filters
- Input is already a dereplicated representative-sequence file heading to chimera removal or MOTU clustering — apply vsearch uchime-denovo or sumaclust instead of dereplicating again
- The goal is taxonomic assignment (ecotag) or in silico PCR primer/probe testing (ecoPCR) — those steps consume the post-clustering MOTU representative sequences, not this intermediate

## Inputs

- Filtered, aligned FASTQ read set (post illuminapairedend quality >40, ngsfilter demultiplexing, ambiguous-base removal, obigrep 120–135 bp length filter), e.g. the Mendeley deposit data.mendeley.com/datasets/489gckxbz3/1

## Outputs

- dereplicated_sequences.fastq — OBITools-annotated FASTQ of representative sequences with per-sample read counts aggregated in the count annotation

## How to apply

Start from the deposited read set produced after initial quality filtering (illuminapairedend alignments with quality score >40 kept), demultiplexing with ngsfilter, ambiguous-base removal, and obigrep length filtering at 120–135 bp. Run obiuniq from the OBITools suite on this filtered FASTQ to collapse all identical sequences into single representative records, aggregating per-sample read counts into the count annotation. Verify the dereplicated output retains sequence identifiers and aggregated abundance annotations in valid OBITools-annotated FASTQ format. Save the result as dereplicated_sequences.fastq so it can feed directly into the vsearch uchime-denovo chimera-removal step and subsequent sumaclust clustering at a constant 99% similarity threshold. Keeping the dereplicated file as an explicitly versioned intermediate mirrors the study's practice of depositing the pre-dereplication file (Mendeley dataset 489gckxbz3/1) so every downstream step is reproducible from it.

## Related tools

- **obiuniq** (Dereplicates the filtered, aligned reads: collapses identical sequences into single records and aggregates per-sample read counts into the count annotation)
- **OBITools** (Metabarcoding software suite providing the annotated-FASTQ format and the obiuniq command used for this intermediate-file step)
- **vsearch** (Downstream consumer of the dereplicated file: runs uchime-denovo chimera removal on the representative sequences)
- **sumaclust** (Downstream consumer of the dereplicated file: delimits MOTUs at a constant 99% similarity threshold)

## Examples

```
obiuniq filtered_reads.fastq > dereplicated_sequences.fastq
```

## Evaluation signals

- Every identical sequence in the input appears exactly once in the output as a single representative record
- The output is valid OBITools-annotated FASTQ retaining sequence identifiers
- Per-sample read counts are aggregated into the count annotation and their totals match the input read abundance
- The file is directly consumable by vsearch uchime-denovo and sumaclust (99% threshold) without reformatting

## Limitations

- The dereplicated intermediate is only as good as its upstream filters — reads passing quality >40, 120–135 bp length and ambiguous-base checks; errors there propagate silently
- Single-read MOTUs derived downstream cannot be fully attributed — contamination or sequencing error cannot be excluded as their cause, so the study removed all single-read MOTUs as a conservative control
- Dereplication discards per-read provenance; only aggregated count annotations survive for later sample-level analyses

## Evidence

- [discussion] deposited read set: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1."
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] chimera removal downstream: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] MOTU clustering downstream: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [discussion] upstream length filter defining the intermediate input: "a length filter (obigrep) was applied to the aligned reads (120–135 bp) in order to select only the fragments with the correct target size."
