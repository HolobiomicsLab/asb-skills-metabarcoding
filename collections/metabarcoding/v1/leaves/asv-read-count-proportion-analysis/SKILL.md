---
name: asv-read-count-proportion-analysis
description: Use when a per-sample-replicate ASV read-count table with ASV sequences
  is available and low-frequency ASV occurrences that closely match a much more abundant
  ASV must be removed as PCR errors before downstream replicate/abundance filtering.
when_to_use_negative:
- The occurrence table has already been through FilterPCRerror (or a denoiser such
  as DADA2/Unoise applied to the same end), so re-filtering would double-remove occurrences
- ASV sequences are unavailable, so a 'closely matching more abundant ASV' cannot
  be identified and the proportion rule cannot be evaluated
- The goal is filtering tag-jumps, contamination, or chimeras — VTAM handles those
  with other filters (FilterChimera, control-based LFN filtering), not FilterPCRerror
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: VTAM
  role: Source pipeline whose FilterPCRerror filter implements the discard rule; can
    be run directly instead of the reimplementation
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language for the pandas-based reimplementation of the filter
    over the benchmark occurrence table
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:1a5f2ce09428fff2bc0a2db5b0d06c46d7de993379390b7682602f4afc69873a
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# asv-read-count-proportion-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies the VTAM FilterPCRerror logic to an ASV occurrence table: discards occurrences of an ASV whose reads are a low proportion of a closely matching, far more abundant ASV, treating them as likely PCR/sequencing errors.

## When to use

Use when a per-sample-replicate ASV read-count table with ASV sequences is available and low-frequency ASV occurrences that closely match a much more abundant ASV must be removed as PCR errors before downstream replicate/abundance filtering.

## When NOT to use

- The occurrence table has already been through FilterPCRerror (or a denoiser such as DADA2/Unoise applied to the same end), so re-filtering would double-remove occurrences
- ASV sequences are unavailable, so a 'closely matching more abundant ASV' cannot be identified and the proportion rule cannot be evaluated
- The goal is filtering tag-jumps, contamination, or chimeras — VTAM handles those with other filters (FilterChimera, control-based LFN filtering), not FilterPCRerror

## Inputs

- ASV occurrence table of read counts per sample-replicate (TSV, e.g. vtam_bat_final_taxa.tsv from the vtam_benchmark output)
- ASV nucleotide sequences (for pairwise similarity matching between candidate error ASVs and more abundant ASVs)
- pcr_error_var_prop threshold (optimized value 0.2 for the bat dataset)

## Outputs

- filtered_asv_occurrences.tsv — TSV of surviving ASV occurrences with PCR-error occurrences removed
- Set of discarded occurrences, each traceable to a closely matching, significantly more abundant ASV

## How to apply

Load the ASV occurrence table (read counts of ASVs per sample-replicate, e.g. vtam_bat_final_taxa.tsv from the VTAM benchmark) with pandas in Python. Compute pairwise similarity among ASV sequences to find, for each ASV, a closely matching ASV with a significantly higher total read count. Apply the FilterPCRerror rule: for each occurrence (ASV i in sample j, replicate k) whose read count is a low proportion of the more abundant match's reads, discard the occurrence when that proportion falls below the pcr_error_var_prop cutoff — 0.2 was the value optimized for the bat dataset. Write surviving occurrences to filtered_asv_occurrences.tsv in TSV format. Rationale: denoising/clustering alone cannot fully remove PCR errors, and VTAM addresses this pitfall explicitly alongside chimeras and pseudogenes via its dedicated filters.

## Related tools

- **VTAM** (Source pipeline whose FilterPCRerror filter implements the discard rule; can be run directly instead of the reimplementation) — https://github.com/aitgon/vtam
- **Python** (Implementation language for the pandas-based reimplementation of the filter over the benchmark occurrence table) — https://github.com/aitgon/vtam

## Examples

```
import pandas as pd; df = pd.read_csv('vtam_bat_final_taxa.tsv', sep='\t'); df[df.read_count >= 0.2 * df.abundant_match_reads].to_csv('filtered_asv_occurrences.tsv', sep='\t', index=False)
```

## Evaluation signals

- filtered_asv_occurrences.tsv exists and parses as a valid TSV with the same sample-replicate columns as vtam_bat_final_taxa.tsv
- Every removed occurrence corresponds to an ASV that closely matches another ASV with significantly higher total reads, matching the VTAM FilterPCRerror definition
- All removed occurrences had read counts below pcr_error_var_prop = 0.2 of the more abundant matching ASV's reads
- No ASV with reads above the pcr_error_var_prop proportion of its more-abundant match is dropped
- Optimized VTAM parameter set is used consistently (lfn_sample_replicate_cutoff 0.002, pcr_error_var_prop 0.2, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60 for the bat dataset)

## Limitations

- The pcr_error_var_prop cutoff of 0.2 was optimized for the bat dataset (Galan et al., 2018); other datasets (fish, shark) used different optimized read-count cut-offs (10 and 40 respectively) and may need different values
- FilterPCRerror only removes occurrences attributed to PCR/sequencing errors; it cannot account for inter-sample contamination, tag-jump, or chimeras, which require VTAM's other filters
- In the VTAM workflow, filter steps are chained (FilterLNF, FilterPCRerror, FilterMinReplicateNumber, FilterRenkonen, FilterChimera, FilterCodonStop, FilterIndel); applying only the PCR-error filter leaves other pitfalls unaddressed

## Evidence

- [readme] VTAM is a Python metabarcoding package producing ASV tables: [evidence span withheld — anchor status: external_doc]
