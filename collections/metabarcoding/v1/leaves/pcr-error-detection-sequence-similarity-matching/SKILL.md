---
name: pcr-error-detection-sequence-similarity-matching
description: Use when an ASV occurrence table contains low-abundance ASVs suspected
  to be PCR/sequencing artifacts of more abundant ASVs, and you need to remove occurrences
  whose read count is a small proportion of a closely matching abundant ASV.
when_to_use_negative:
- The occurrence table has already passed a denoising/clustering algorithm (e.g. DADA2,
  Unoise, Deblur, Swarm) that explicitly models PCR/sequencing errors — re-running
  similarity-based error removal risks discarding genuine rare variants.
- The goal is to remove chimeras, pseudogenes, or indel-containing spurious sequences,
  which VTAM handles with the distinct FilterChimera, FilterCodonStop, and FilterIndel
  filters.
- The occurrence read count is not a small proportion of a closely matching more abundant
  ASV — such occurrences are not FilterPCRerror candidates and should instead be addressed
  by replicate-based filters (FilterMinReplicateNumber, FilterRenkonen).
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: VTAM
  role: Pipeline whose filter command implements FilterPCRerror, discarding occurrences
    of ASVs closely matching a more abundant ASV
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM and of the pandas-based reconstruction/loading
    of the ASV occurrence table
  repo: https://github.com/aitgon/vtam
- name: vsearch
  role: Dependency installed for VTAM, usable for pairwise sequence similarity computation
    among ASVs
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:7bf96ba8fc02ed8085a8dc3e15047f162b4132b44d1f24b8c61bf9afb37167be
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
  - vsearch
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pcr-error-detection-sequence-similarity-matching

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reconstructs VTAM's FilterPCRerror step, which discards ASV occurrences that closely match another ASV with a significantly higher read count, treating them as likely PCR/sequencing errors. In the bat-dataset benchmark this uses a pcr_error_var_prop cutoff of 0.2.

## When to use

Use when an ASV occurrence table contains low-abundance ASVs suspected to be PCR/sequencing artifacts of more abundant ASVs, and you need to remove occurrences whose read count is a small proportion of a closely matching abundant ASV.

## When NOT to use

- The occurrence table has already passed a denoising/clustering algorithm (e.g. DADA2, Unoise, Deblur, Swarm) that explicitly models PCR/sequencing errors — re-running similarity-based error removal risks discarding genuine rare variants.
- The goal is to remove chimeras, pseudogenes, or indel-containing spurious sequences, which VTAM handles with the distinct FilterChimera, FilterCodonStop, and FilterIndel filters.
- The occurrence read count is not a small proportion of a closely matching more abundant ASV — such occurrences are not FilterPCRerror candidates and should instead be addressed by replicate-based filters (FilterMinReplicateNumber, FilterRenkonen).

## Inputs

- ASV occurrence table with read counts per ASV per sample-replicate (TSV, e.g. vtam_bat_final_taxa.tsv from the VTAM benchmark)
- ASV sequences (for computing pairwise similarity between variants)

## Outputs

- filtered_asv_occurrences.tsv — TSV of surviving ASV occurrences after FilterPCRerror removal

## How to apply

Load the ASV occurrence table (read counts of ASVs per sample-replicate, e.g. from the benchmark output vtam_bat_final_taxa.tsv) with pandas in Python. For each ASV, compute pairwise sequence similarity to identify a closely matching ASV with a significantly higher total number of reads. Apply FilterPCRerror logic: for each occurrence (ASV i in sample j, replicate k) whose read count is a low proportion of the closely matching more abundant ASV's reads, discard the occurrence when that proportion falls below the pcr_error_var_prop cutoff — 0.2 was the value optimized for the bat dataset in the VTAM benchmark (alongside lfn_sample_replicate_cutoff 0.002, lfn_variant_cutoff 0.006, lfn_read_count_cutoff 60). Write surviving occurrences to a TSV (filtered_asv_occurrences.tsv) and validate that removed occurrences correspond exclusively to ASVs closely matching a more abundant ASV below the cutoff. The rationale is that denoising/clustering alone cannot handle all pitfalls, so VTAM pairs this variant-level filter with other filters (FilterLNF, FilterMinReplicateNumber, FilterRenkonen, FilterChimera, FilterCodonStop, FilterIndel) using control samples to optimize thresholds.

## Related tools

- **VTAM** (Pipeline whose filter command implements FilterPCRerror, discarding occurrences of ASVs closely matching a more abundant ASV) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM and of the pandas-based reconstruction/loading of the ASV occurrence table) — https://github.com/aitgon/vtam
- **vsearch** (Dependency installed for VTAM, usable for pairwise sequence similarity computation among ASVs) — https://github.com/aitgon/vtam

## Examples

```
python -c "import pandas as pd; df = pd.read_csv('vtam_bat_final_taxa.tsv', sep='\t'); df[df.read_count / df.abundant_match_reads >= 0.2].to_csv('filtered_asv_occurrences.tsv', sep='\t', index=False)"
```

## Evaluation signals

- filtered_asv_occurrences.tsv exists and parses as a valid TSV.
- Every removed occurrence corresponds to an ASV that closely matches another ASV with a significantly higher number of reads.
- For each removed occurrence (ASV i in sample j, replicate k), the read count proportion relative to the closely matching abundant ASV is below pcr_error_var_prop = 0.2 (optimized for the bat dataset).
- Surviving occurrences are unchanged for ASVs with no closely matching more abundant ASV.
- Removed occurrences account exclusively for PCR-error candidates, not for chimera, pseudogene, or non-reproducible-replicate removals (handled by other VTAM filters).

## Limitations

- The pcr_error_var_prop cutoff is dataset-specific: 0.2 was optimized for the bat dataset; the fish and shark datasets had different suggested read-count cut-offs (10 and 40), so the threshold must be re-optimized per dataset.
- Existing denoising/clustering algorithms filter PCR and sequencing errors but cannot account for inter-sample contamination, tag-jump, and chimeras — this skill must be combined with other tools/filters for those pitfalls.
- Discarding low-proportion occurrences risks false negatives if genuinely rare, biologically real ASVs closely resemble abundant ASVs.

## Evidence

- [intro] VTAM addresses pitfalls including PCR errors: [evidence span withheld — anchor status: not_found]
