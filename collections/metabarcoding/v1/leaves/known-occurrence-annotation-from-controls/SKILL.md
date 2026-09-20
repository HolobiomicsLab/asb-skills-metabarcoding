---
name: known-occurrence-annotation-from-controls
description: Use when running the VTAM pipeline on metabarcoding data with mock community
  and negative control samples and you need to annotate true/false-positive occurrences
  to feed parameter optimization.
when_to_use_negative:
- Your data lack control samples (mock communities and/or negative controls) — VTAM's
  optimization is explicitly designed to use them and cannot derive the cutoffs without
  them
- You are analyzing non-coding 16S amplicons where FilterCodonStop and FilterIndel
  must be skipped, so the standard coding-marker filter chain does not apply unchanged
- You already have a validated ASV table and only need taxonomic assignment (use VTAM's
  taxassign instead of re-optimizing filters)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: VTAM
  role: Provides the make_known_occurrences, filter, and optimize commands used to
    annotate control-derived true/false positives and optimize cutoffs
  repo: https://github.com/aitgon/vtam
- name: Python
  role: VTAM is implemented in Python and installed via pip into a conda environment
  repo: https://github.com/aitgon/vtam
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:9a4960e566e06145c92623ff6ed509d0a69bc76baf799e38fbde426688804c01
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# known-occurrence-annotation-from-controls

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Annotate ASV occurrences as true positives (present in mock community samples) or false positives (present only in mock and negative controls) using VTAM's make_known_occurrences command, producing the known_occurrences.tsv file that drives VTAM's optimize step.

## When to use

Use when running the VTAM pipeline on metabarcoding data with mock community and negative control samples and you need to annotate true/false-positive occurrences to feed parameter optimization.

## When NOT to use

- Your data lack control samples (mock communities and/or negative controls) — VTAM's optimization is explicitly designed to use them and cannot derive the cutoffs without them
- You are analyzing non-coding 16S amplicons where FilterCodonStop and FilterIndel must be skipped, so the standard coding-marker filter chain does not apply unchanged
- You already have a validated ASV table and only need taxonomic assignment (use VTAM's taxassign instead of re-optimizing filters)

## Inputs

- ASV occurrence table (output of VTAM filter run with default parameters)
- Sample/replicate metadata (FASTA info) identifying mock community samples and negative controls
- Expected taxon composition of mock community samples
- Dataset-specific FASTQ amplicon data (COI or 16S markers)

## Outputs

- known_occurrences.tsv listing true positives (mock samples) and false positives (mock and negative controls)
- Per-dataset optimized parameter file with lfn_sample_replicate_cutoff, pcr_error_var_prop, lfn_variant_cutoff, and lfn_read_count_cutoff

## How to apply

First run VTAM's filter command with default parameters on the merged, demultiplexed ASV occurrence tables (from merge and sortreads) for your dataset, e.g. the bat, fish (MFZR and ZFZR markers separately), or shark datasets. Then run the make_known_occurrences command on the filter output to produce known_occurrences.tsv, marking occurrences as true positives when expected taxa appear in mock samples and as false positives when unexpected variants appear in mock or negative control samples. This annotated file is the explicit control-driven input to VTAM's optimize command: lfn_sample_replicate_cutoff and pcr_error_var_prop are optimized first (independently of other parameters), then lfn_variant_cutoff and lfn_read_count_cutoff are chosen as the compromise minimizing false positives and false negatives. For non-coding markers such as 16S in the shark dataset, skip FilterCodonStop and FilterIndel when preparing the data. The resulting per-dataset optima were lfn_read_count_cutoff 60 (bat), 10 (fish), and 40 (shark), alongside lfn_sample_replicate_cutoff/pcr_error_var_prop/lfn_variant_cutoff values of 0.002/0.2/0.006 (bat), 0.003/0.1/0.006 (fish), and 0.001/0.2/0.016 (shark).

## Related tools

- **VTAM** (Provides the make_known_occurrences, filter, and optimize commands used to annotate control-derived true/false positives and optimize cutoffs) — https://github.com/aitgon/vtam
- **Python** (VTAM is implemented in Python and installed via pip into a conda environment) — https://github.com/aitgon/vtam

## Examples

```
vtam make_known_occurrences
```

## Evaluation signals

- known_occurrences.tsv exists and lists occurrences flagged as true positives in mock samples and false positives in mock and negative controls
- The emitted per-dataset parameter file contains exactly four optimized values matching the reported optima (e.g. lfn_read_count_cutoff 60 for bat, 10 for fish, 40 for shark)
- lfn_sample_replicate_cutoff and pcr_error_var_prop are optimized before lfn_variant_cutoff and lfn_read_count_cutoff, consistent with their independence from the other parameters
- For the shark (16S) dataset, FilterCodonStop and FilterIndel are absent from the filter chain, and the optimized cutoffs differ (0.001/0.2/0.016/40)
- Filtering with the optimized parameters yields clean mock and negative control samples (no false positives) while retaining expected mock taxa (no false negatives)

## Limitations

- Optimization depends on adequate mock communities; commercial mock samples exist for bacteria and fungi, but metabarcoders studying metazoa or plants must construct adequate, representative mock samples
- The optimum is a compromise between false positives and false negatives across parameter combinations, not a globally unique setting
- Software versions and dataset accession numbers are not fully specified in the published methods section

## Evidence

- [readme] VTAM is a metabarcoding package for HTS amplicon data: [evidence span withheld — anchor status: external_doc]
