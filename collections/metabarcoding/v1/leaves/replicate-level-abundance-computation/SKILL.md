---
name: replicate-level-abundance-computation
description: Use when you have an ASV-by-sample-by-replicate occurrence table with
  read counts and need replicate-level abundance values to apply low-frequency-noise
  (LNF) cutoffs before filtering occurrences.
when_to_use_negative:
- Input is already a filtered occurrence table or an ASV table without replicate-level
  read counts, so Nijk/Njk and Nijk/Ni cannot be computed
- The analysis goal is PCR-error removal rather than low-frequency-noise removal —
  FilterPCRerror discards occurrences by ASV match/read-count comparisons, not by
  abundance cutoffs
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0632
- http://edamontology.org/topic_3308
tools:
- name: VTAM
  role: Implements the filter step including FilterLNF, which applies the computed
    abundance cutoffs to eliminate occurrences
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Language in which VTAM (and custom cutoff-application scripts) are implemented
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:49bf9865261f3aaa7e9ba98b03642d307d756d0ba864b9bf931e2cc06bf59d0e
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0632
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# replicate-level-abundance-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-occurrence abundance metrics (absolute read count Nijk, relative abundance within sample-replicate Nijk/Njk, and relative abundance within ASV Nijk/Ni) from the ASV occurrence table, which underpin VTAM's FilterLNF low-frequency-noise removal.

## When to use

Use when you have an ASV-by-sample-by-replicate occurrence table with read counts and need replicate-level abundance values to apply low-frequency-noise (LNF) cutoffs before filtering occurrences.

## When NOT to use

- Input is already a filtered occurrence table or an ASV table without replicate-level read counts, so Nijk/Njk and Nijk/Ni cannot be computed
- The analysis goal is PCR-error removal rather than low-frequency-noise removal — FilterPCRerror discards occurrences by ASV match/read-count comparisons, not by abundance cutoffs

## Inputs

- ASV occurrence table with columns ASV ID, sample, replicate, and read count (Nijk)
- Per-sample-replicate read totals (Njk)
- Per-ASV read totals (Ni)
- LNF cutoff parameters: lfn_read_count_cutoff, lfn_sample_replicate_cutoff, lfn_variant_cutoff

## Outputs

- Filtered occurrence table (e.g., filtered_occurrences.tsv) retaining ASV ID, sample, replicate, and read count for occurrences passing all three LNF cutoffs

## How to apply

Load the ASV occurrence table indexed by (ASV i, sample j, replicate k) with read counts Nijk, per-sample-replicate totals Njk, and per-ASV totals Ni. For each occurrence compute three quantities: absolute reads Nijk, relative abundance within the sample-replicate Nijk/Njk, and relative abundance within the ASV Nijk/Ni. Flag or remove an occurrence if Nijk < lfn_read_count_cutoff OR Nijk/Njk < lfn_sample_replicate_cutoff OR Nijk/Ni < lfn_variant_cutoff; the VTAM-optimized bat-dataset values are lfn_read_count_cutoff=60, lfn_sample_replicate_cutoff=0.002, lfn_variant_cutoff=0.006. Write surviving occurrences (ASV ID, sample, replicate, read count) to a filtered occurrences table. This computation is the quantitative core of the FilterLNF step that eliminates low-frequency-noise occurrences alongside FilterPCRerror.

## Related tools

- **VTAM** (Implements the filter step including FilterLNF, which applies the computed abundance cutoffs to eliminate occurrences) — https://github.com/aitgon/vtam
- **Python** (Language in which VTAM (and custom cutoff-application scripts) are implemented) — https://github.com/aitgon/vtam

## Examples

```
python -c "import pandas as pd; t=pd.read_csv('occurrences.tsv',sep='\t'); keep=(t.read_count>=60)&(t.read_count/t.total_reads_per_sample_replicate>=0.002)&(t.read_count/t.total_reads_per_asv>=0.006); t[keep].to_csv('filtered_occurrences.tsv',sep='\t',index=False)"
```

## Evaluation signals

- No occurrence in the output violates any cutoff: all retained occurrences have Nijk >= 60, Nijk/Njk >= 0.002, and Nijk/Ni >= 0.006 (bat-dataset values)
- Output file is a well-formed tab-separated table preserving ASV ID, sample, replicate and read count columns
- Every removed occurrence failed at least one of the three LNF criteria (absolute reads, relative abundance within sample-replicate, relative abundance within ASV)
- Per-sample-replicate totals Njk and per-ASV totals Ni used in the ratios match the input occurrence table

## Limitations

- Cutoff values are dataset- and marker-specific: VTAM optimized lfn_read_count_cutoff=60 for the bat dataset, but suggested cut-offs were 10 for fish and 40 for shark, so thresholds must be re-optimized per dataset rather than reused
- VTAM explicitly processes negative control and mock samples to determine optimal filtering parameter settings; without such control samples, the optimized thresholds may not minimize both false negatives and false positives

## Evidence
