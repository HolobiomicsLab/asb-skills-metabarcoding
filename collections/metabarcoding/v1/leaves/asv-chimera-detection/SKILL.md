---
name: asv-chimera-detection
description: Use when a FASTQ-derived ASV occurrence table and merged fasta from VTAM's
  merge/sortreads steps must be cleaned of chimeric sequences before downstream filtering
  or taxonomic assignment.
when_to_use_negative:
- Input is already a chimera-filtered ASV table or comes from a denoising pipeline
  (DADA2, Unoise, Deblur) that already handles PCR/sequencing errors differently
- Input is raw FASTQ reads that have not been merged, demultiplexed and trimmed by
  VTAM's merge and sortreads commands
- The goal is filtering non-reproducible occurrences (FilterMinReplicateNumber) or
  whole replicates (FilterRenkonen) rather than chimeras
edam_operation: http://edamontology.org/operation_3227
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_0080
tools:
- name: VTAM
  role: Provides the 'filter' command with the FilterChimera filter that eliminates
    chimeric ASVs
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM and its filter step
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:25df43045ffcd4257fbc63b4e5f015ccda735c7286146b8c58ec2ccc7fe00afa
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/aitgon/vtam_benchmark
  repo_url: https://github.com/aitgon/vtam
  tools:
  - Python
  - VTAM
  edam_operation: http://edamontology.org/operation_3227
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# ASV chimera detection and removal

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Detects and eliminates chimeric ASVs from metabarcoding occurrence tables using VTAM's FilterChimera, which compares each ASV against more abundant ASVs, alongside pseudogene filters (FilterCodonStop, FilterIndel).

## When to use

Use when a FASTQ-derived ASV occurrence table and merged fasta from VTAM's merge/sortreads steps must be cleaned of chimeric sequences before downstream filtering or taxonomic assignment.

## When NOT to use

- Input is already a chimera-filtered ASV table or comes from a denoising pipeline (DADA2, Unoise, Deblur) that already handles PCR/sequencing errors differently
- Input is raw FASTQ reads that have not been merged, demultiplexed and trimmed by VTAM's merge and sortreads commands
- The goal is filtering non-reproducible occurrences (FilterMinReplicateNumber) or whole replicates (FilterRenkonen) rather than chimeras

## Inputs

- ASV occurrence table (ASV i in sample j of replicate k with read count Nijk)
- Merged fasta sequence file from VTAM merge/sortreads steps
- VTAM filter configuration with FilterChimera enabled

## Outputs

- Chimera-filtered occurrence table (TSV) with all occurrences of chimeric ASVs removed
- List of removed chimeric ASV sequence IDs

## How to apply

Load the ASV occurrence table (ASV i in sample j of replicate k with read count Nijk) together with the merged fasta sequences produced by VTAM's merge and sortreads commands (e.g. on the bat dataset of Galan et al. 2018, Dryad doi:10.5061/dryad.kv02g). Run the VTAM 'filter' command — implemented in Python3 within the VTAM/WopMars framework — with FilterChimera enabled so that each ASV is compared against more abundant ASVs to flag chimeric sequences; FilterCodonStop and FilterIndel remove pseudogenes and spurious sequences, and other filters can be deactivated for a standalone chimera operation. Extract occurrences flagged as chimeras and record their sequence IDs as the removed chimeric ASV list. Write the chimera-filtered occurrence table (all occurrences of chimeric ASVs removed) as a TSV file.

## Related tools

- **VTAM** (Provides the 'filter' command with the FilterChimera filter that eliminates chimeric ASVs) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM and its filter step) — https://github.com/aitgon/vtam

## Examples

```
vtam filter --fastqfile merged.fasta --asvtable asvtable.tsv --fastafile sorted_reads.fasta --min_replicate_number 1 --filter_chimera
```

## Evaluation signals

- No ASV flagged as a chimera remains in the filtered occurrence table
- The set of removed ASV IDs exactly matches the chimera-flag list
- All occurrences of each chimeric ASV (across samples and replicates) are absent from the output TSV
- Non-chimeric ASVs and their read counts (Nijk) are unchanged between input and filtered tables

## Limitations

- Existing denoising/clustering algorithms alone cannot account for inter-sample contamination, tag-jump, and chimeras and must be combined with other tools
- FilterChimera operates alongside FilterCodonStop and FilterIndel, so chimera removal alone does not remove pseudogenes and spurious sequences
- Software versions of the compared tools were not stated in the publication's benchmark sections

## Evidence

- [methods] Chimeras are eliminated by the FilterChimera, and pseudogenes and spurious sequences by FilterCodonStop and FilterIndel: "Chimeras are eliminated by the FilterChimera, and pseudogenes and spurious sequences by FilterCodonStop and FilterIndel"
- [intro] they cannot account for inter-sample contamination, tag-jump, and chimeras and therefore must be combined with other tools: "they cannot account for inter-sample contamination, tag-jump, and chimeras and therefore must be combined with other tools"
- [methods] The output fasta files are the input data for the filter step of VTAM: "The output fasta files are the input data for the filter step of VTAM"
- [methods] merged, demultiplexed and trimmed from tags and primers by VTAM using the merge and sortreads commands: "merged, demultiplexed and trimmed from tags and primers by VTAM using the merge and sortreads commands"
- [readme] VTAM is a metabarcoding package with various commands to process high throughput sequencing (HTS) data of amplicons: [evidence span withheld — anchor status: external_doc]
