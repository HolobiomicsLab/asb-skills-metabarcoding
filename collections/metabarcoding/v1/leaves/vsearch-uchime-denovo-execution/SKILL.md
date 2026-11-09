---
name: vsearch-uchime-denovo-execution
description: Use when processing dereplicated amplicon reads (e.g. obiuniq output
  from filtered eDNA reads) and a chimera-free sequence set is required before MOTU
  clustering, with no suitable reference database available for reference-based chimera
  checking.
when_to_use_negative:
- Input reads have not yet been dereplicated — uchime-denovo's de novo model depends
  on abundance information, so run quality/length filtering and obiuniq dereplication
  first
- Sequences are already assigned to MOTUs or a feature table exists — chimera removal
  must occur upstream of sumaclust MOTU delimitation
- A high-quality reference database is available and reference-based chimera detection
  is preferred for closely related taxa
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: vsearch
  role: Implements the uchime-denovo algorithm used to identify and remove chimeric
    sequences de novo from the dereplicated reads
- name: obiuniq
  role: Produces the dereplicated sequence input (with read abundances) on which uchime-denovo
    is run
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-017-17150-2
    title: Environmental DNA reveals tropical shark diversity in contrasting levels
      of anthropogenic impact
  source_content_hash: sha256:00e70529f128fd136a4e106512965c967acab994db213d61ff06410ba625d06e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - obiuniq
  - vsearch
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# vsearch-uchime-denovo-execution

## Summary

Executes the uchime-denovo algorithm implemented in vsearch to identify and remove chimeric amplicon sequences de novo (without a reference database) from dereplicated eDNA metabarcoding reads, so that a chimera-filtered FASTA is carried forward to MOTU delimitation.

## When to use

Use when processing dereplicated amplicon reads (e.g. obiuniq output from filtered eDNA reads) and a chimera-free sequence set is required before MOTU clustering, with no suitable reference database available for reference-based chimera checking.

## When NOT to use

- Input reads have not yet been dereplicated — uchime-denovo's de novo model depends on abundance information, so run quality/length filtering and obiuniq dereplication first
- Sequences are already assigned to MOTUs or a feature table exists — chimera removal must occur upstream of sumaclust MOTU delimitation
- A high-quality reference database is available and reference-based chimera detection is preferred for closely related taxa

## Inputs

- Dereplicated FASTA file of eDNA amplicon reads produced by obiuniq (from quality- and length-filtered reads)
- Dereplicated reads with abundance information (PCR chimera detection relies on read abundance patterns)

## Outputs

- FASTA file containing non-chimeric sequences, ready for MOTU clustering (e.g. sumaclust at 99% similarity)

## How to apply

First, ensure the input is the dereplicated sequence file produced by obiuniq from the length- and quality-filtered paired-end reads (in this pipeline: illumimapairedend alignments with quality score >40 kept, obigrep length filter of 120–135 bp, ambiguous bases removed). Then run vsearch's uchime-denovo algorithm in de novo mode, which detects chimeras purely from the abundances and compositional relationships within the dereplicated read set, without any reference database. Export the non-chimeric sequences to a FASTA output file and discard all sequences flagged as chimeric by uchime-denovo. The chimera-filtered FASTA then feeds directly into MOTU delimitation (sumaclust at a 99% similarity threshold in this study) and subsequent taxonomic assignment with ecotag.

## Related tools

- **vsearch** (Implements the uchime-denovo algorithm used to identify and remove chimeric sequences de novo from the dereplicated reads)
- **obiuniq** (Produces the dereplicated sequence input (with read abundances) on which uchime-denovo is run)

## Examples

```
vsearch --uchime_denovo dereplicated.fasta --nonchimeras chimera_filtered.fasta
```

## Evaluation signals

- The chimera-filtered output FASTA exists and is a valid FASTA file
- No sequences in the output FASTA were flagged as chimeric by uchime-denovo
- Output contains only sequences present in the input dereplicated file (subset relationship; dereplicated headers/abundances preserved)
- The pipeline order is respected: chimera removal occurs after dereplication and before sumaclust MOTU delimitation

## Limitations

- De novo chimera detection relies on abundance and composition within the sample set; without a reference database, rare chimeras may escape detection
- Chimera removal alone does not remove spurious single-read MOTUs: this study additionally removed all single-read MOTUs because contamination or sequencing error could not be excluded
- This step does not address other sequence artifacts such as primer bias or taxonomic resolution limits of the 127 bp COI fragment

## Evidence

- [discussion] a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77: "a chimera removal step was performed using the uchime-denovo algorithm76 implemented in vsearch77"
- [discussion] The reads were subsequently dereplicated using obiuniq: "The reads were subsequently dereplicated using obiuniq"
- [discussion] The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%.: "The MOTUs were delimited using the sumaclust algorithm75 with a constant similarity threshold of 99%."
- [readme] Reference databases in fasta format for Eukaryotic metabarcoding: "Reference databases in fasta format for Eukaryotic metabarcoding"
