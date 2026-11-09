---
name: amplicon-sequence-chimera-detection
description: Use when dereplicated amplicon FASTA reads (e.g. after illuminapairedend
  alignment, quality/length filtering and obiuniq dereplication) must be screened
  for chimeric sequences before MOTU clustering, without a reference database.
when_to_use_negative:
- Input is raw paired-end FASTQ or a non-dereplicated read set — the workflow requires
  dereplicated input from obiuniq after quality and length filtering
- A curated, comprehensive reference database exists and reference-based chimera checking
  (e.g. uchime against references) is preferred; this skill is specifically the de
  novo (uchime-denovo) route
- Input is already a MOTU table or post-clustering representative sequences — chimera
  removal must precede sumaclust MOTU delimitation
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: vsearch
  role: Implements the uchime-denovo algorithm used to identify chimeric sequences
    de novo from dereplicated reads
- name: obiuniq
  role: Produces the dereplicated sequence file that serves as input to the chimera
    removal step
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:2c6b34caa9da70c430cc5613c318348f2a3776fcd04ffdfe2365ca22c3901234
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - obiuniq
  - vsearch
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# amplicon-sequence-chimera-detection

## Summary

Detects and removes chimeric PCR amplicons de novo from dereplicated eDNA metabarcoding reads using the uchime-denovo algorithm implemented in vsearch, so that a chimera-free sequence set is carried forward to MOTU delimitation with sumaclust.

## When to use

Use when dereplicated amplicon FASTA reads (e.g. after illuminapairedend alignment, quality/length filtering and obiuniq dereplication) must be screened for chimeric sequences before MOTU clustering, without a reference database.

## When NOT to use

- Input is raw paired-end FASTQ or a non-dereplicated read set — the workflow requires dereplicated input from obiuniq after quality and length filtering
- A curated, comprehensive reference database exists and reference-based chimera checking (e.g. uchime against references) is preferred; this skill is specifically the de novo (uchime-denovo) route
- Input is already a MOTU table or post-clustering representative sequences — chimera removal must precede sumaclust MOTU delimitation

## Inputs

- Dereplicated FASTA file of eDNA amplicon reads produced by obiuniq (after illuminapairedend alignment, quality score >40 filtering, 120–135 bp length filtering, and removal of reads with ambiguous bases)

## Outputs

- Chimera-filtered FASTA file containing only non-chimeric sequences, carried forward to sumaclust MOTU clustering at 99% similarity

## How to apply

Load the dereplicated sequence file produced by obiuniq from the length- (120–135 bp, obigrep) and quality- (>40) filtered eDNA COI amplicon reads. Run uchime-denovo as implemented in vsearch in de novo mode, i.e. without any reference database, so chimeras are detected from abundance and topology patterns among the reads themselves. Export the non-chimeric sequences to a FASTA file and discard sequences flagged as chimeric. The chimera-filtered FASTA then feeds directly into MOTU delimitation with sumaclust at a 99% similarity threshold and subsequent ecotag taxonomic assignment. De novo detection is appropriate here because the bespoke elasmobranch reference database (BOLD plus ecoPCR in silico PCR products from EMBL-EBI release R117) may not cover all true novel diversity, making reference-based chimera checking risky.

## Related tools

- **vsearch** (Implements the uchime-denovo algorithm used to identify chimeric sequences de novo from dereplicated reads)
- **obiuniq** (Produces the dereplicated sequence file that serves as input to the chimera removal step)

## Examples

```
vsearch --uchime_denovo dereplicated.fasta --nonchimeras chimera_filtered.fasta
```

## Evaluation signals

- Confirm the chimera-filtered FASTA exists and is a valid FASTA file
- Verify no sequences flagged as chimeric by uchime-denovo remain in the output file
- Check the output is a strict subset of the dereplicated input (read counts should only decrease)
- Downstream sanity check: MOTUs delimited by sumaclust at 99% similarity should derive exclusively from the chimera-filtered sequence set

## Limitations

- De novo chimera detection relies on read abundance patterns and may miss low-abundance chimeras; the paper additionally removed all single-read MOTUs as a conservative contamination/false-positive control, acknowledging that contamination or sequencing error cannot be completely excluded
- Chimera filtering occurs within a broader trade-off between taxonomic resolution and non-target amplification that the authors note must be balanced before conservation decisions

## Evidence

- [discussion] a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] we have opted for a more conservative approach and have removed all single read MOTUs from our samples: "we have opted for a more conservative approach and have removed all single read MOTUs from our samples."
