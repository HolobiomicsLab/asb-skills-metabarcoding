---
name: obitools-sequence-dereplication
description: Use when you have an OBITools-annotated FASTQ of filtered, aligned eDNA
  reads (e.g. 120–135 bp COI fragments) and need to collapse identical sequences into
  representative records with aggregated sample-level abundance counts prior to chimera
  detection or clustering.
when_to_use_negative:
- Input is already a dereplicated representative-sequence file or a MOTU table (e.g.
  sumaclust output) — re-running obiuniq would not add information.
- Input reads have not yet been length-filtered or demultiplexed (per the article's
  pipeline, dereplication follows illuminapairedend alignment, ngsfilter demultiplexing
  and obigrep 120–135 bp filtering).
- The goal is chimera removal — that is performed separately with the vsearch uchime-denovo
  algorithm on the dereplicated file, not by obiuniq.
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: obiuniq
  role: Dereplicates the filtered read file by collapsing identical sequences into
    single records and aggregating per-sample read counts into the count annotation
- name: OBITools
  role: Metabarcoding software suite providing obiuniq and the annotated FASTQ format
    used for the dereplicated output
- name: obigrep
  role: Upstream filter producing the 120–135 bp length-selected read set that is
    the direct input to dereplication
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:eaf0fc1d84a12cedeb2cb1849b7025f691353de3b1934310ce5dad5406f1a2c9
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OBITools
  - obigrep
  - obiuniq
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 6
---

# obitools-sequence-dereplication

## Summary

Dereplicate filtered, demultiplexed eDNA metabarcoding reads (post-illuminapairedend alignment, ngsfilter demultiplexing and obigrep 120–135 bp length filtering) by collapsing identical sequences into single records with the OBITools obiuniq tool, aggregating per-sample read counts into the count annotation before chimera removal and MOTU clustering.

## When to use

Use when you have an OBITools-annotated FASTQ of filtered, aligned eDNA reads (e.g. 120–135 bp COI fragments) and need to collapse identical sequences into representative records with aggregated sample-level abundance counts prior to chimera detection or clustering.

## When NOT to use

- Input is already a dereplicated representative-sequence file or a MOTU table (e.g. sumaclust output) — re-running obiuniq would not add information.
- Input reads have not yet been length-filtered or demultiplexed (per the article's pipeline, dereplication follows illuminapairedend alignment, ngsfilter demultiplexing and obigrep 120–135 bp filtering).
- The goal is chimera removal — that is performed separately with the vsearch uchime-denovo algorithm on the dereplicated file, not by obiuniq.

## Inputs

- OBITools-annotated FASTQ of filtered, aligned, demultiplexed reads (120–135 bp COI fragments, quality score >40, ambiguous bases removed), e.g. from https://data.mendeley.com/datasets/489gckxbz3/1

## Outputs

- dereplicated_sequences.fastq: OBITools-annotated FASTQ of collapsed representative sequences with per-sample read counts aggregated in the count annotation

## How to apply

Obtain the deposited read set (post quality filtering at quality score >40 with illuminapairedend, demultiplexing with ngsfilter, ambiguous-base removal, and obigrep length filtering to 120–135 bp) from the Mendeley repository. Run obiuniq from the OBITools suite on the filtered FASTQ so that all identical sequences collapse into single records, aggregating per-sample read counts into the count annotation (use the -m count option so per-sample abundances are merged correctly). Verify the dereplicated output is valid OBITools-annotated FASTQ that retains sequence identifiers and the aggregated abundance annotations. Save the result as dereplicated_sequences.fastq for consumption by the downstream vsearch uchime-denovo chimera-removal step and sumaclust 99% similarity MOTU clustering.

## Related tools

- **obiuniq** (Dereplicates the filtered read file by collapsing identical sequences into single records and aggregating per-sample read counts into the count annotation)
- **OBITools** (Metabarcoding software suite providing obiuniq and the annotated FASTQ format used for the dereplicated output)
- **obigrep** (Upstream filter producing the 120–135 bp length-selected read set that is the direct input to dereplication)

## Examples

```
obiuniq -m count filtered_reads.fastq > dereplicated_sequences.fastq
```

## Evaluation signals

- All identical sequences are collapsed: each distinct sequence appears exactly once in the output FASTQ.
- The count annotation on each output record equals the sum of that sequence's per-sample read counts across the input file.
- The output is valid OBITools-annotated FASTQ that retains sequence identifiers and aggregated abundance annotations.
- The total number of records decreases substantially relative to the input while total aggregated read counts are conserved.
- The dereplicated file is accepted downstream by vsearch uchime-denovo chimera removal and sumaclust 99% similarity clustering without format errors.

## Limitations

- Dereplication does not remove chimeras or sequencing errors; those are handled downstream (uchime-denovo in vsearch) and by removing single-read MOTUs as a conservative contamination/false-positive control.
- Some closely related shark species (e.g. Carcharhinus, Rhizoprionodon, Negaprion) cannot be separated at the 127 bp COI fragment, so some dereplicated representative sequences are 100% identical to multiple species in BOLD.
- Certain elasmobranch species present at sampling (e.g. Ginglymostoma cirratum) were not recovered at all due to primer mismatches, so absence in the dereplicated file does not imply absence in the environment.

## Evidence

- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] a length filter (obigrep) was applied to the aligned reads (120–135 bp): "a length filter (obigrep) was applied to the aligned reads (120–135 bp) in order to select only the fragments with the correct target size."
- [discussion] Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at Mendeley: "Original DNA sequences (after initial quality filtering, paired-end alignment and length filtering) can be found at: https://data.mendeley.com/datasets/489gckxbz3/1."
- [readme] Reference databases README describing the metabarpark repository and citation for COI elasmobranch sequences: "Reference databases in fasta format for Eukaryotic metabarcoding"
