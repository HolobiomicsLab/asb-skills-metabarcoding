---
name: edna-amplicon-read-processing
description: Use when a FASTQ/annotated read set from paired-end Illumina amplicon
  sequencing (post-alignment, demultiplexing and length filtering) must be collapsed
  to unique representative sequences prior to chimera detection and MOTU clustering.
when_to_use_negative:
- Input reads have not yet been aligned, demultiplexed, or length-filtered — dereplicating
  raw or unfiltered reads would collapse artefactual variants prematurely
- The data is already a MOTU table or feature table — dereplication operates on raw
  sequence reads, not abundance matrices
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3168
tools:
- name: obiuniq
  role: Dereplicates the filtered read file, collapsing identical sequences into single
    records with aggregated count annotations
- name: OBITools
  role: Metabarcoding software suite providing obiuniq and the annotated-FASTQ format
    used for the dereplicated output
- name: obigrep
  role: Upstream length filter (120–135 bp) applied to aligned reads before dereplication
- name: illuminapairedend
  role: Upstream paired-end alignment step whose quality >40 alignments form the input
    to the filtered read set
- name: ngsfilter
  role: Upstream demultiplexing of the aligned dataset prior to dereplication
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:1f72939b123ebb42a84f867e95580b16fedd3468d3895767ac456eef535dbd48
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - illuminapairedend
  - ngsfilter
  - obigrep
  - obiuniq
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# edna-amplicon-read-processing

## Summary

Dereplication of quality- and length-filtered eDNA amplicon reads using obiuniq from the OBITools suite, collapsing identical sequences into single records with aggregated per-sample abundance annotations before chimera removal and MOTU clustering.

## When to use

Use when a FASTQ/annotated read set from paired-end Illumina amplicon sequencing (post-alignment, demultiplexing and length filtering) must be collapsed to unique representative sequences prior to chimera detection and MOTU clustering.

## When NOT to use

- Input reads have not yet been aligned, demultiplexed, or length-filtered — dereplicating raw or unfiltered reads would collapse artefactual variants prematurely
- The data is already a MOTU table or feature table — dereplication operates on raw sequence reads, not abundance matrices

## Inputs

- FASTQ file of filtered, aligned reads (post illuminapairedend alignment at quality >40, ngsfilter demultiplexing, ambiguous-base removal, obigrep 120–135 bp length filter), e.g. deposited at data.mendeley.com/datasets/489gckxbz3/1

## Outputs

- Dereplicated representative-sequence file in OBITools-annotated FASTQ format (dereplicated_sequences.fastq) with aggregated per-sample count annotations, consumed by vsearch uchime-denovo and sumaclust

## How to apply

Take the deposited read file produced after paired-end alignment with illuminapairedend (keeping alignments with quality score >40), demultiplexing with ngsfilter, removal of reads with ambiguous bases, and obigrep length filtering to 120–135 bp. Run obiuniq from the OBITools metabarcoding suite on this file so that all identical sequences collapse into single records, with per-sample read counts aggregated into the count annotation, yielding a dereplicated OBITools-annotated FASTQ (e.g. dereplicated_sequences.fastq). Verify the output retains sequence identifiers and aggregated abundance annotations in valid annotated FASTQ format before feeding it to vsearch uchime-denovo chimera removal and sumaclust clustering at a 99% similarity threshold. Dereplication is placed before chimera detection so that downstream steps operate on representative sequences rather than redundant reads.

## Related tools

- **obiuniq** (Dereplicates the filtered read file, collapsing identical sequences into single records with aggregated count annotations)
- **OBITools** (Metabarcoding software suite providing obiuniq and the annotated-FASTQ format used for the dereplicated output)
- **obigrep** (Upstream length filter (120–135 bp) applied to aligned reads before dereplication)
- **illuminapairedend** (Upstream paired-end alignment step whose quality >40 alignments form the input to the filtered read set)
- **ngsfilter** (Upstream demultiplexing of the aligned dataset prior to dereplication)

## Examples

```
obiuniq -m count filtered_reads.fastq > dereplicated_sequences.fastq
```

## Evaluation signals

- Dereplicated output is valid OBITools-annotated FASTQ and retains sequence identifiers
- Per-sample read counts are aggregated into the count annotation of each unique record
- The dereplicated file (dereplicated_sequences.fastq) is accepted by the downstream vsearch uchime-denovo chimera-removal and sumaclust 99% clustering steps
- Every sequence in the output is unique (no duplicate sequence strings remain)
- Total reads across count annotations equal the read count of the filtered input file

## Limitations

- Dereplication is only as good as the upstream filters: reads with ambiguous bases or incorrect lengths (outside 120–135 bp) must be removed first
- Single-read MOTUs surviving into downstream analysis cannot be attributed — contamination or sequencing error cannot be completely excluded as their cause
- Primer bias and varying shedding rates mean aggregated read counts are not a validated proxy for species biomass

## Evidence

- [methods] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [methods] Paired-end alignment and quality thresholds: "Paired-end reads were aligned using illuminapairedend and alignments with quality score >40 were kept."
- [methods] Length filter defining the input read set: "a length filter (obigrep) was applied to the aligned reads (120–135 bp) in order to select only the fragments with the correct target size."
- [methods] Deposited filtered read set used as dereplication input: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1."
- [methods] Downstream consumption of dereplicated representative sequences: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
