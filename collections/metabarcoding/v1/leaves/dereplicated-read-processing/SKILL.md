---
name: dereplicated-read-processing
description: Use when dereplicated amplicon FASTA files (e.g. produced by obiuniq
  after illuminapairedend alignment, quality-score >40 filtering and 120–135 bp length
  filtering) may contain chimeric sequences that must be removed without a reference
  database before MOTU clustering.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been aligned (illuminapairedend), quality-filtered,
  length-filtered and dereplicated — chimera detection should operate on the dereplicated
  dataset, not raw reads
- Sequences are already clustered into MOTUs or a feature/abundance table has been
  generated — chimera removal must occur before sumaclust delimitation
- A curated reference-based chimera checking strategy is specifically required, since
  uchime-denovo is a de novo method performed without a reference database
edam_operation: http://edamontology.org/operation_3287
edam_topics:
- http://edamontology.org/topic_0637
- http://edamontology.org/topic_0196
tools:
- name: vsearch
  role: Implements the uchime-denovo algorithm used to detect and remove chimeric
    sequences de novo from the dereplicated read dataset
- name: obiuniq
  role: Produces the dereplicated sequence input file that the chimera removal step
    operates on
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:d8ca56919f80ef1ed6fd9f736d8cce0956871fc568fbcf267d38044a29623a14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - obiuniq
  - vsearch
  edam_operation: http://edamontology.org/operation_3287
  edam_topics:
  - http://edamontology.org/topic_0637
  - http://edamontology.org/topic_0196
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# dereplicated-read-processing

## Summary

Chimera filtering of dereplicated eDNA metabarcoding reads using the de novo uchime-denovo algorithm implemented in vsearch, so that a chimera-filtered sequence dataset is carried forward to MOTU delimitation (sumaclust, 99% similarity threshold).

## When to use

Use when dereplicated amplicon FASTA files (e.g. produced by obiuniq after illuminapairedend alignment, quality-score >40 filtering and 120–135 bp length filtering) may contain chimeric sequences that must be removed without a reference database before MOTU clustering.

## When NOT to use

- Input is raw FASTQ reads that have not yet been aligned (illuminapairedend), quality-filtered, length-filtered and dereplicated — chimera detection should operate on the dereplicated dataset, not raw reads
- Sequences are already clustered into MOTUs or a feature/abundance table has been generated — chimera removal must occur before sumaclust delimitation
- A curated reference-based chimera checking strategy is specifically required, since uchime-denovo is a de novo method performed without a reference database

## Inputs

- Dereplicated FASTA file of quality- and length-filtered (120–135 bp) eDNA amplicon reads produced by obiuniq

## Outputs

- Chimera-filtered FASTA file of non-chimeric sequences
- Set of sequences flagged as chimeric by uchime-denovo (discarded)

## How to apply

Load the dereplicated sequence file produced by obiuniq from the length- (120–135 bp) and quality-filtered eDNA amplicon reads. Run uchime-denovo, as implemented in vsearch, in de novo mode: it detects chimeric sequences without any reference database by comparing sequences against each other within the dataset. Export only the non-chimeric sequences to a FASTA output file, discarding reads flagged as chimeric. The rationale in Bakker et al. (2017) is that PCR chimeras inflate MOTU counts, so this step must precede sumaclust clustering at the 99% similarity threshold and subsequent ecotag taxonomic assignment. This de novo mode is appropriate for COI eDNA data where a trusted chimera reference set for the elasmobranch 127 bp 12S/COI amplicons is unavailable.

## Related tools

- **vsearch** (Implements the uchime-denovo algorithm used to detect and remove chimeric sequences de novo from the dereplicated read dataset)
- **obiuniq** (Produces the dereplicated sequence input file that the chimera removal step operates on)

## Examples

```
vsearch --uchime_denovo dereplicated_reads.fasta --nonchimeras chimera_filtered.fasta
```

## Evaluation signals

- Chimera-filtered FASTA output file exists and is a valid FASTA file
- Output contains no sequences flagged as chimeric by uchime-denovo
- The chimera-filtered dataset is carried forward intact to sumaclust MOTU delimitation at the 99% similarity threshold
- Input is the dereplicated obiuniq output derived from reads passing the >40 quality score and 120–135 bp length filters, confirming the correct upstream pipeline order
- Number of sequences in the chimera-filtered FASTA is less than or equal to the dereplicated input (only chimeras removed)

## Limitations

- uchime-denovo works without a reference database, so its sensitivity depends on de novo detection within the sample dataset rather than curated reference chimeras
- Only chimeric sequences are removed; other error sources such as contamination or sequencing error (e.g. single-read MOTUs) are handled separately by a conservative downstream step that removes all single read MOTUs
- The 127 bp COI fragment limits taxonomic resolution downstream — some MOTUs were 100% identical to multiple BOLD species — which affects interpretation but not the chimera removal itself

## Evidence

- [discussion] a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] a length filter (obigrep) was applied to the aligned reads (120–135 bp) in order to select only the fragments with the correct target size.: "a length filter (obigrep) was applied to the aligned reads (120–135 bp) in order to select only the fragments with the correct target size."
- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
