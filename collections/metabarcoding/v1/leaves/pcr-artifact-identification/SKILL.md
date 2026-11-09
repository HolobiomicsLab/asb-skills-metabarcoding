---
name: pcr-artifact-identification
description: Use when a merged ASV fasta plus occurrence table (ASV i in sample j
  of replicate k with read count Nijk) from VTAM merge/sortreads may contain chimeras
  or pseudogenes that must be flagged and eliminated before abundance analysis.
when_to_use_negative:
- Input is raw FASTQ reads that have not yet been merged, demultiplexed, and trimmed
  by VTAM merge/sortreads — FilterChimera operates on the filter step's fasta inputs
- The goal is filtering PCR or sequencing errors or low-frequency noise rather than
  chimeras/pseudogenes — FilterLNF and FilterPCRerror address those occurrences instead
- Non-reproducible occurrences or whole replicates are the target — FilterMinReplicateNumber
  and FilterRenkonen handle those, not FilterChimera
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0080
tools:
- name: VTAM
  role: Provides the filter step with the FilterChimera, FilterCodonStop, and FilterIndel
    filters that detect and eliminate chimeras, pseudogenes, and spurious sequences
    from ASV occurrence tables
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM in which the chimera-detection filter logic
    runs
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:4d0c6aa4f6a75090236cb973166877f9b085e8203de1923d676e3b18c27e3f94
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
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# pcr-artifact-identification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identifies and removes PCR-derived artifacts — chimeric ASVs, pseudogenes, and spurious indel-containing sequences — from metabarcoding ASV tables using VTAM's FilterChimera, FilterCodonStop, and FilterIndel filters.

## When to use

Use when a merged ASV fasta plus occurrence table (ASV i in sample j of replicate k with read count Nijk) from VTAM merge/sortreads may contain chimeras or pseudogenes that must be flagged and eliminated before abundance analysis.

## When NOT to use

- Input is raw FASTQ reads that have not yet been merged, demultiplexed, and trimmed by VTAM merge/sortreads — FilterChimera operates on the filter step's fasta inputs
- The goal is filtering PCR or sequencing errors or low-frequency noise rather than chimeras/pseudogenes — FilterLNF and FilterPCRerror address those occurrences instead
- Non-reproducible occurrences or whole replicates are the target — FilterMinReplicateNumber and FilterRenkonen handle those, not FilterChimera

## Inputs

- Merged fasta sequences produced by VTAM merge/sortreads commands (e.g. from the bat dataset, Galan et al. 2018, Dryad doi:10.5061/dryad.kv02g)
- ASV occurrence table (ASV i in sample j of replicate k with read count Nijk)

## Outputs

- Chimera-flag list (sequence IDs of ASVs flagged as chimeric)
- Chimera-filtered occurrence table written as a TSV file with all occurrences of chimeric ASVs removed

## How to apply

Run the VTAM 'filter' step (Python3, within the VTAM/WopMars framework) on the fasta files output by the merge and sortreads commands, with FilterChimera enabled so each ASV is compared against more abundant ASVs to detect chimeric sequences; FilterCodonStop and FilterIndel are run alongside to remove pseudogenes and spurious sequences. For a standalone chimera-reconstruction task, other filters (e.g. FilterLNF, FilterPCRerror, FilterMinReplicateNumber, FilterRenkonen) can be deactivated while keeping FilterChimera active. Extract the occurrences flagged as chimeras from the filter output and record their sequence IDs as the removed chimeric ASV list. Write the chimera-filtered occurrence table as a TSV with all occurrences of chimeric ASVs removed. Rationale: denoising/clustering algorithms such as DADA2, Unoise, Deblur, and Swarm cannot account for chimeras, so an explicit chimera-detection filter must be combined with them.

## Related tools

- **VTAM** (Provides the filter step with the FilterChimera, FilterCodonStop, and FilterIndel filters that detect and eliminate chimeras, pseudogenes, and spurious sequences from ASV occurrence tables) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM in which the chimera-detection filter logic runs) — https://github.com/aitgon/vtam

## Evaluation signals

- No ASV flagged as a chimera remains in the filtered occurrence table
- The set of removed ASV IDs exactly matches the chimera-flag list
- Chimera-filtered TSV row counts equal the input occurrence counts minus all occurrences (across samples and replicates) of flagged ASVs
- Filtered occurrence table is a strict subset of the input Nijk occurrence table, with unflagged ASVs unchanged
- Pseudogene/spurious-sequence removal by FilterCodonStop and FilterIndel can be traced in the filter output for ASVs containing codon stops or indels

## Limitations

- Chimera detection compares each ASV against more abundant ASVs, so rare chimeras without an abundant parent may be missed
- Denoising/clustering algorithms cannot account for inter-sample contamination, tag-jump, and chimeras and therefore must be combined with other tools — chimera filtering alone does not remove these other artifact classes
- Software versions used in the benchmark are not stated in the extracted sections

## Evidence

- [intro] VTAM addresses PCR errors, chimeras, and pseudogenes: [evidence span withheld — anchor status: not_found]
- [readme] VTAM scope from README: [evidence span withheld — anchor status: external_doc]
