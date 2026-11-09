---
name: metabarcoding-read-count-normalization
description: Use when you have an ASV occurrence table with read counts Nijk (ASV
  i, sample j, replicate k) from HTS amplicon data plus negative controls, and you
  need to remove low-frequency noise occurrences before downstream replicate-consistency
  or taxonomic-assignment steps.
when_to_use_negative:
- When the occurrence-level noise to remove is a sequencing/PCR error closely matching
  a high-read ASV — that is the job of FilterPCRerror, not the LFN cutoffs
- When occurrences have not yet been produced by the upstream VTAM filter step (filter,
  sortreads, merge) — LFN cutoffs operate on ASV occurrence tables, not raw FASTQ
- When filtering should be driven by per-taxon abundance patterns across control samples
  — that is handled by VTAM's optimize/make_known_occurrences parameter optimization,
  not the LFN cutoffs
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_0091
tools:
- name: VTAM
  role: Provides the FilterLNF filter and the filter command whose output occurrence
    tables are filtered by the LFN cutoffs
  repo: https://github.com/aitgon/vtam
- name: Python
  role: Implementation language of VTAM, used to script the LFN cutoff computation
    on the occurrence table
  repo: https://github.com/aitgon/vtam
- name: WopMars
  role: Workflow manager used to orchestrate VTAM processing steps reproducibly
  repo: github.com/aitgon/wopmars
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:8deea65f1e2004ece1713077dfca254638be49375a62dc2bfe86fdfece8ca7dc
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
  - WopMars
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# metabarcoding-read-count-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies VTAM's FilterLNF (Filter Low Frequency Noise) logic, which removes low-frequency-noise occurrences from an ASV-by-sample-replicate read-count table using three cutoffs (absolute reads, within-sample-replicate relative abundance, and within-ASV relative abundance) to eliminate false positives such as tag-jump and low-level contamination that denoisers like DADA2 or Unoise cannot address.

## When to use

Use when you have an ASV occurrence table with read counts Nijk (ASV i, sample j, replicate k) from HTS amplicon data plus negative controls, and you need to remove low-frequency noise occurrences before downstream replicate-consistency or taxonomic-assignment steps.

## When NOT to use

- When the occurrence-level noise to remove is a sequencing/PCR error closely matching a high-read ASV — that is the job of FilterPCRerror, not the LFN cutoffs
- When occurrences have not yet been produced by the upstream VTAM filter step (filter, sortreads, merge) — LFN cutoffs operate on ASV occurrence tables, not raw FASTQ
- When filtering should be driven by per-taxon abundance patterns across control samples — that is handled by VTAM's optimize/make_known_occurrences parameter optimization, not the LFN cutoffs

## Inputs

- ASV occurrence table with read counts Nijk indexed by ASV, sample, and replicate
- Per-sample-replicate read totals Njk
- Per-ASV read totals Ni
- Optimized LFN parameter values (lfn_read_count_cutoff, lfn_sample_replicate_cutoff, lfn_variant_cutoff)

## Outputs

- Filtered occurrence table (e.g. filtered_occurrences.tsv) retaining only occurrences passing all three LFN cutoffs, with ASV ID, sample, replicate and read count columns

## How to apply

Load the ASV occurrence table (ASV i, sample j, replicate k) with per-occurrence read counts Nijk, per-sample-replicate totals Njk, and per-ASV totals Ni. For each occurrence compute three quantities: absolute reads Nijk, relative abundance within the sample-replicate Nijk/Njk, and relative abundance within the ASV Nijk/Ni. Remove the occurrence if Nijk < lfn_read_count_cutoff OR Nijk/Njk < lfn_sample_replicate_cutoff OR Nijk/Ni < lfn_variant_cutoff; in the VTAM benchmark on the bat dataset (357 fecal pellet samples, 24 mock samples, 19 negative controls, 3 PCR replicates each) the optimized values were lfn_read_count_cutoff=60, lfn_sample_replicate_cutoff=0.002, and lfn_variant_cutoff=0.006. The rationale is that occurrences failing these cutoffs are low-frequency noise (e.g. tag-jump or inter-sample contamination) rather than true biology, since VTAM explicitly uses control samples to determine thresholds that minimize both false negatives and false positives. Write surviving occurrences to a filtered table preserving ASV ID, sample, replicate and read count columns, and combine with the companion FilterPCRerror step that discards ASVs closely matching a much higher-read ASV.

## Related tools

- **VTAM** (Provides the FilterLNF filter and the filter command whose output occurrence tables are filtered by the LFN cutoffs) — https://github.com/aitgon/vtam
- **Python** (Implementation language of VTAM, used to script the LFN cutoff computation on the occurrence table) — https://github.com/aitgon/vtam
- **WopMars** (Workflow manager used to orchestrate VTAM processing steps reproducibly) — github.com/aitgon/wopmars

## Evaluation signals

- No occurrence in the output table violates any cutoff: every retained row has Nijk >= 60, Nijk/Njk >= 0.002, and Nijk/Ni >= 0.006 (bat-dataset values)
- Output is a well-formed tab-separated table preserving ASV ID, sample, replicate and read count columns
- Known false-positive occurrences previously observed in negative controls are absent after filtering
- True positives (species expected in mock community samples) are retained, indicating the cutoffs did not introduce false negatives
- The output feeds correctly into downstream filters (FilterMinReplicateNumber, FilterRenkonen) and make_known_occurrences without schema errors

## Limitations

- Cutoff values are dataset- and marker-specific: VTAM's suggested read-count cut-offs were 10 for fish, 40 for shark, and 60 for bat datasets, so thresholds must be re-optimized per dataset via control samples
- The optimization of lfn_sample_replicate_cutoff and pcr_error_var_prop is independent of the other parameters, so parameter sweeps must respect this separability
- Denoising/clustering algorithms (DADA2, Unoise, Deblur, Swarm) cannot account for inter-sample contamination, tag-jump, and chimeras — LFN filtering mitigates but does not replace the need for control-based parameter optimization

## Evidence

- [readme] VTAM processes HTS amplicon data into an ASV table: [evidence span withheld — anchor status: external_doc]
