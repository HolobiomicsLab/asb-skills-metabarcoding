---
name: chimera-filtered-fasta-output-handling
description: Use when dereplicated amplicon reads (post quality/length filtering)
  must be screened for chimeric PCR products before MOTU clustering and no curated
  chimera reference database is available.
when_to_use_negative:
- Input reads are not yet dereplicated — uchime-denovo operates on the dereplicated
  read set produced by obiuniq, not raw paired-end or demultiplexed reads
- A curated chimera reference database is available and reference-based chimera checking
  is preferred — uchime-denovo is specifically the de novo (reference-free) mode
- Input is already a MOTU table or clustered at 99% similarity — chimera removal must
  precede sumaclust clustering, not follow it
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_0652
- http://edamontology.org/topic_0080
tools:
- name: vsearch
  role: Implements the uchime-denovo de novo chimera detection algorithm executed
    on the dereplicated reads
- name: obiuniq
  role: Produces the dereplicated sequence file that serves as the input to chimera
    removal
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:b06c045d9f0ae65ee6a9e1fb609a27ed81dbc25b736a8322a3371ac3dc5baa14
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - obiuniq
  - vsearch
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_0652
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# chimera-filtered-fasta-output-handling

## Summary

Removes chimeric amplicon sequences de novo from dereplicated eDNA metabarcoding reads using the uchime-denovo algorithm implemented in vsearch, exporting a chimera-filtered FASTA that is carried forward to MOTU delimitation (sumaclust at 99% similarity).

## When to use

Use when dereplicated amplicon reads (post quality/length filtering) must be screened for chimeric PCR products before MOTU clustering and no curated chimera reference database is available.

## When NOT to use

- Input reads are not yet dereplicated — uchime-denovo operates on the dereplicated read set produced by obiuniq, not raw paired-end or demultiplexed reads
- A curated chimera reference database is available and reference-based chimera checking is preferred — uchime-denovo is specifically the de novo (reference-free) mode
- Input is already a MOTU table or clustered at 99% similarity — chimera removal must precede sumaclust clustering, not follow it

## Inputs

- Dereplicated FASTA file of amplicon reads (output of obiuniq, after length filtering to 120–135 bp and removal of reads with ambiguous bases)

## Outputs

- Chimera-filtered FASTA file containing only non-chimeric sequences
- Set/flagged list of sequences identified as chimeras by uchime-denovo (discarded)

## How to apply

Load the dereplicated sequence file produced by obiuniq from the length-filtered (120–135 bp), quality-filtered (alignment quality score >40) eDNA reads. Run uchime-denovo in de novo mode via vsearch, which identifies chimeras from the read set itself without any reference database. Export the non-chimeric sequences to a FASTA output file and discard sequences flagged as chimeric. In the Bakker et al. (2017) eDNA shark metabarcoding pipeline, this chimera-filtered dataset is the direct input to MOTU delimitation with sumaclust at a 99% similarity threshold, so the chimera step must complete before clustering.

## Related tools

- **vsearch** (Implements the uchime-denovo de novo chimera detection algorithm executed on the dereplicated reads)
- **obiuniq** (Produces the dereplicated sequence file that serves as the input to chimera removal)

## Examples

```
vsearch --uchime_denovo dereplicated.fasta --nonchimeras chimera_filtered.fasta
```

## Evaluation signals

- The chimera-filtered FASTA exists and is a valid FASTA file
- The output FASTA contains no sequences flagged as chimeric by uchime-denovo
- Every sequence in the chimera-filtered FASTA is present in the input dereplicated read set (output is a subset of input)
- The chimera-filtered FASTA is produced before MOTU delimitation with sumaclust at the 99% similarity threshold
- Sequences removed (chimeras) can be enumerated and account for the difference between input and output read counts

## Limitations

- uchime-denovo detects chimeras de novo without a reference database, so chimera detection depends solely on the structure of the dereplicated read set itself
- Chimera removal precedes MOTU clustering, but downstream contamination or sequencing error cannot be fully excluded — the authors note it is not possible to completely exclude contamination (or sequencing error) as the potential cause of single-read MOTUs

## Evidence

- [discussion] a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
