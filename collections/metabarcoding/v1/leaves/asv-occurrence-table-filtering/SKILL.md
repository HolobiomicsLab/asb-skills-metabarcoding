---
name: asv-occurrence-table-filtering
description: Use when a FASTQ-derived ASV occurrence table (read counts per ASV, sample
  and PCR replicate) must be stringently filtered to remove false positives such as
  PCR errors, chimeras, low-frequency noise, tag-jump artifacts, and false negatives
  from non-reproducible detections.
when_to_use_negative:
- When the input is already a filtered, validated ASV table — reapplying occurrence
  filters would discard legitimate low-abundance detections.
- When the goal is denoising or clustering of raw reads (Swarm, Unoise, Deblur, DADA2)
  rather than occurrence-level validation of ASVs using control samples.
- When no negative controls, mock samples or PCR replicates are available, since VTAM's
  cutoff optimization explicitly depends on these experimental design elements.
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3174
tools:
- name: VTAM
  role: Implements the filter step with FilterLNF, FilterPCRerror, FilterMinReplicateNumber,
    FilterRenkonen, FilterChimera, FilterCodonStop and FilterIndel, plus the optimize
    command to set cutoffs from control samples
  repo: https://github.com/aitgon/vtam
- name: Python
  role: VTAM implementation language; used to load occurrence tables with pandas,
    apply cutoffs and write TSV outputs during benchmark reconstruction
- name: WopMars
  role: Workflow manager used to run VTAM reproducibly
  repo: https://github.com/aitgon/wopmars/
merged_aliases:
- asv-occurrence-filtering-metabarcoding
merged_alias_records:
- alias: asv-occurrence-filtering-metabarcoding
  slug: asv-occurrence-filtering-metabarcoding
  jaccard_score: 0.6
  method: token-set-jaccard
  decision: needs_review
provenance:
  source_task_ids:
  - task_006
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2023.01.034
    title: 'VTAM: A robust pipeline for validating metabarcoding data using controls'
  source_content_hash: sha256:2211e47223cebd78fce62ba8ed99c613fc32b090131ed4b7e4a1baafc8ded871
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
  - http://edamontology.org/topic_3174
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# ASV occurrence table filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Apply VTAM's occurrence-level filters (FilterLNF, FilterPCRerror, FilterMinReplicateNumber, FilterRenkonen, FilterChimera, FilterCodonStop/FilterIndel) to an ASV × sample × replicate read-count table, using control-sample-optimized cutoffs (e.g. lfn_read_count_cutoff=60, lfn_sample_replicate_cutoff=0.002, lfn_variant_cutoff=0.006, pcr_error_var_prop=0.2 for the bat dataset) to remove PCR errors, low-frequency noise, non-reproducible occurrences, chimeras and pseudogenes while minimizing false positives and false negatives.

## When to use

Use when a FASTQ-derived ASV occurrence table (read counts per ASV, sample and PCR replicate) must be stringently filtered to remove false positives such as PCR errors, chimeras, low-frequency noise, tag-jump artifacts, and false negatives from non-reproducible detections.

## When NOT to use

- When the input is already a filtered, validated ASV table — reapplying occurrence filters would discard legitimate low-abundance detections.
- When the goal is denoising or clustering of raw reads (Swarm, Unoise, Deblur, DADA2) rather than occurrence-level validation of ASVs using control samples.
- When no negative controls, mock samples or PCR replicates are available, since VTAM's cutoff optimization explicitly depends on these experimental design elements.

## Inputs

- ASV occurrence table with read counts Nijk per ASV i, sample j, PCR replicate k (e.g. vtam_bat_final_taxa.tsv, TSV format)
- Per-sample-replicate read totals Njk and per-ASV read totals Ni
- FASTA files output by VTAM merge and sortreads commands
- Negative control, mock community and replicate samples from the experimental design (e.g. bat dataset: 357 fecal pellet samples, 24 mock samples, 19 negative controls, 3 PCR replicates each)

## Outputs

- Filtered occurrence table (e.g. filtered_occurrences.tsv / filtered_asv_occurrences.tsv) preserving ASV ID, sample, replicate and read count columns
- Final validated ASV table ready for taxonomic assignment (taxassign) and downstream analysis

## How to apply

Feed the FASTA files produced by VTAM's merge and sortreads commands into the filter step, then apply the occurrence-level filters in sequence. For FilterLNF, remove an occurrence (ASV i, sample j, replicate k, read count Nijk) when Nijk < lfn_read_count_cutoff OR Nijk/Njk < lfn_sample_replicate_cutoff OR Nijk/Ni < lfn_variant_cutoff (bat-dataset optimized values: 60, 0.002 and 0.006). For FilterPCRerror, discard an ASV occurrence when the ASV closely matches another ASV with a significantly higher number of reads and its read proportion falls below pcr_error_var_prop (0.2). Apply FilterMinReplicateNumber to drop non-reproducible occurrences and FilterRenkonen to drop whole replicates, then FilterChimera, FilterCodonStop and FilterIndel to eliminate chimeras, pseudogenes and spurious sequences. Cutoffs should be optimized using negative control and mock samples (VTAM's optimize command) so that the settings producing clean controls are applied to real samples.

## Related tools

- **VTAM** (Implements the filter step with FilterLNF, FilterPCRerror, FilterMinReplicateNumber, FilterRenkonen, FilterChimera, FilterCodonStop and FilterIndel, plus the optimize command to set cutoffs from control samples) — https://github.com/aitgon/vtam
- **Python** (VTAM implementation language; used to load occurrence tables with pandas, apply cutoffs and write TSV outputs during benchmark reconstruction)
- **WopMars** (Workflow manager used to run VTAM reproducibly) — https://github.com/aitgon/wopmars/

## Examples

```
cd example; snakemake --printshellcmds --resources db=1 --snakefile snakefile.yml --cores 4 --configfile asper1/user_input/snakeconfig_mfzr.yml --until asvtable_taxa
```

## Evaluation signals

- No occurrence in filtered_occurrences.tsv violates any cutoff: all retained rows satisfy Nijk >= 60, Nijk/Njk >= 0.002 and Nijk/Ni >= 0.006 for the bat-dataset parameters.
- Output file parses as a well-formed tab-separated table preserving ASV ID, sample, replicate and read count columns.
- Removed occurrences correspond exclusively to ASVs closely matching a more abundant ASV below the pcr_error_var_prop=0.2 proportion, consistent with the FilterPCRerror definition.
- Filtered output from negative controls and mock samples is clean (no unexpected taxa), matching the optimization objective of minimizing false positives.
- Precision of the filtered dataset improves relative to unfiltered baselines, as observed for VTAM compared to DALU and OBIbaR.

## Limitations

- Cutoffs are dataset-specific: VTAM suggested read-count cut-offs were 10 for fish, 40 for shark and 60 for bat datasets, so values must be re-optimized per dataset.
- Denoising/clustering algorithms alone cannot account for inter-sample contamination, tag-jump and chimeras, so occurrence filtering must complement them rather than replace them.
- Metabarcoding data remain prone to false negatives and false positives if filtering thresholds are not correctly addressed.
- Careful construction of mock communities is required; commercial mock samples exist only for some taxonomic groups such as bacteria and fungi.

## Evidence

- [methods] The FilterLNF (Filter Low Frequency Noise) and FilterPCRerror filters eliminate occurrences: "The FilterLNF (Filter Low Frequency Noise) and FilterPCRerror filters eliminate occurrences"
- [methods] FilterPCRerror definition: "The FilterPCRerror discards occurrences if an ASV closely matches another ASV with a significantly higher number of reads"
- [methods] Replicate filters: "Two filters eliminate non-reproducible occurrences (FilterMinReplicateNumber), or whole replicates (FilterRenkonen)"
- [methods] Chimera/pseudogene filters: "Chimeras are eliminated by the FilterChimera, and pseudogenes and spurious sequences by FilterCodonStop and FilterIndel"
- [readme] VTAM scope from README: [evidence span withheld — anchor status: external_doc]
