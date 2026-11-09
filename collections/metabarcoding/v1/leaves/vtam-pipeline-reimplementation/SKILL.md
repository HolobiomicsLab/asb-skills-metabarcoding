---
name: vtam-pipeline-reimplementation
description: Use when you have an ASV occurrence table (read counts per sample-replicate,
  e.g. vtam_bat_final_taxa.tsv) and need to remove occurrences attributable to PCR
  errors by comparing closely matching, more abundant ASVs.
when_to_use_negative:
- When the target pitfall is low-frequency noise rather than PCR errors — use VTAM's
  FilterLNF (Filter Low Frequency Noise) instead
- When the occurrences are non-reproducible across replicates or whole replicates
  fail the Renkonen test — use FilterMinReplicateNumber or FilterRenkonen instead
- When the input is raw FASTQ reads rather than an ASV occurrence table — VTAM's merge
  and sortreads commands must run first to produce the filter step's fasta inputs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: VTAM
  role: Source pipeline whose FilterPCRerror filter is being reimplemented; discards
    occurrences if an ASV closely matches another ASV with a significantly higher
    number of reads
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Language used (via pandas) to load the ASV occurrence table and re-implement
    the FilterPCRerror logic with the pcr_error_var_prop cutoff
  repo: https://github.com/aitgon/vtam
- name: Swarm
  role: Referenced denoising/clustering algorithm that, like Unoise/DADA2, filters
    most PCR and sequencing errors but cannot account for contamination, tag-jump,
    and chimeras — motivating this occurrence-level filter
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:dc61823d42d935fd6c5d1e5edb30c166034c3bee4b47a78e2b2c977ba348f202
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
  - Swarm
  - VTAM
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# vtam-pipeline-reimplementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reimplements the VTAM FilterPCRerror filter, which discards ASV occurrences whenever the ASV closely matches another ASV with a significantly higher number of reads, treating it as a likely PCR/sequencing error. This addresses one of the methodological pitfalls of eDNA metabarcoding (PCR errors) within VTAM's control-based, variant-based filtering pipeline.

## When to use

Use when you have an ASV occurrence table (read counts per sample-replicate, e.g. vtam_bat_final_taxa.tsv) and need to remove occurrences attributable to PCR errors by comparing closely matching, more abundant ASVs.

## When NOT to use

- When the target pitfall is low-frequency noise rather than PCR errors — use VTAM's FilterLNF (Filter Low Frequency Noise) instead
- When the occurrences are non-reproducible across replicates or whole replicates fail the Renkonen test — use FilterMinReplicateNumber or FilterRenkonen instead
- When the input is raw FASTQ reads rather than an ASV occurrence table — VTAM's merge and sortreads commands must run first to produce the filter step's fasta inputs

## Inputs

- ASV occurrence table with read counts of ASVs per sample-replicate (TSV, e.g. vtam_bat_final_taxa.tsv from the VTAM benchmark)
- ASV sequences for computing pairwise similarity between variants

## Outputs

- filtered_asv_occurrences.tsv — TSV table of ASV occurrences surviving the FilterPCRerror filter

## How to apply

Load the ASV occurrence table (read counts of ASVs per sample-replicate) from the benchmark output vtam_bat_final_taxa.tsv using pandas in Python. Compute pairwise similarity among ASV sequences to identify, for each ASV, a closely matching ASV with a significantly higher total number of reads. Apply FilterPCRerror: for each occurrence (ASV i in sample j, replicate k) whose read count is a low proportion of the closely matching more abundant ASV's reads, discard the occurrence when that proportion falls below the pcr_error_var_prop cutoff — 0.2, the value optimized for the bat dataset alongside lfn_sample_replicate_cutoff 0.002, lfn_variant_cutoff 0.006, and lfn_read_count_cutoff 60. The rationale, per the VTAM paper, is that denoising/clustering algorithms alone cannot account for inter-sample contamination, tag-jump, and chimeras, so VTAM pairs them with explicit occurrence-level filters using control samples and replicates. Write the surviving occurrences to filtered_asv_occurrences.tsv in TSV format and validate that removed occurrences correspond exclusively to ASVs closely matching a more abundant ASV below the pcr_error_var_prop proportion.

## Related tools

- **VTAM** (Source pipeline whose FilterPCRerror filter is being reimplemented; discards occurrences if an ASV closely matches another ASV with a significantly higher number of reads) — https://github.com/aitgon/vtam
- **Python** (Language used (via pandas) to load the ASV occurrence table and re-implement the FilterPCRerror logic with the pcr_error_var_prop cutoff) — https://github.com/aitgon/vtam
- **Swarm** (Referenced denoising/clustering algorithm that, like Unoise/DADA2, filters most PCR and sequencing errors but cannot account for contamination, tag-jump, and chimeras — motivating this occurrence-level filter)

## Examples

```
python filter_pcr_error.py --input vtam_bat_final_taxa.tsv --output filtered_asv_occurrences.tsv --pcr_error_var_prop 0.2
```

## Evaluation signals

- filtered_asv_occurrences.tsv exists and parses as a valid TSV
- Every removed occurrence corresponds to an ASV that closely matches another ASV with a significantly higher number of reads
- The read count of each discarded occurrence is below the pcr_error_var_prop = 0.2 proportion of the closely matching more abundant ASV's reads
- No occurrences of ASVs lacking a closely matching, more abundant variant are removed
- The optimized bat dataset parameters (pcr_error_var_prop 0.2, lfn_sample_replicate_cutoff 0.002, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60) are used when reproducing the bat dataset benchmark

## Limitations

- pcr_error_var_prop = 0.2 is the value optimized for the bat dataset (Galan et al., 2018, 357 bat fecal pellet samples); other datasets may require different cutoffs
- FilterPCRerror removes only PCR-error-like occurrences; it does not address chimeras (FilterChimera), pseudogenes (FilterCodonStop), indels (FilterIndel), or non-reproducible occurrences (FilterMinReplicateNumber, FilterRenkonen)
- Requires a closely matching ASV with significantly higher reads to exist; isolated erroneous ASVs will not be caught by this filter alone

## Evidence

- [readme] VTAM purpose from README: [evidence span withheld — anchor status: external_doc]
